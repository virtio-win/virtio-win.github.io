**(This page is under construction)**

All the operations required for DMA transfer must be implemented via WDF DMA API and not via legacy routines for physical memory manipulation (otherwise the device driver is not compatible with IOMMU and VIRTIO_F_ACCESS_PLATFORM feature). These operations include:
* Allocation of physical memory suitable for DMA transfer
* Obtaining physical/virtual addresses of allocated memory blocks
* Mapping of existing memory (for example buffers of WDF requests) in order to use it for DMA transfers

Virtio-WDF library includes necessary procedures to simplify implementation of DMA transfer between guest driver and virtio PCI device.

## Initialization

No special initialization for DMA operations.
When the driver calls VirtIOWdfInitialize (typically from EvtDevicePrepareHardware procedure) the library initializes also WDFDMAENABLER object for the device. The library uses it during device lifecycle for DMA operations.

## Clean-up:

No special clean-up for DMA operations.
When the driver calls VirtIOWdfShutdown (typically from EvtDeviceReleaseHardware) the library destroys also WDFDMAENABLER object

## Allocation of contiguous memory buffer suitable for DMA transfer

Use this method of allocation when you need memory blocks of one or more 4K pages

``void *VirtIOWdfDeviceAllocDmaMemory(VirtIODevice *vdev, size_t size, ULONG groupTag)``

**size** - block size to allocate

**groupTag** - optional value that the driver can use for memory allocation. All the DMA buffers allocated with the same non-zero tag can be freed together, see **VirtIOWdfDeviceFreeDmaMemoryByTag** below

Return value:

in case of error: NULL

in case of success: virtual address of the beginning of the allocated memory block

**IRQL: PASSIVE**

## Obtaining physical address of the location inside allocated memory block:

``PHYSICAL_ADDRESS VirtIOWdfDeviceGetPhysicalAddress(VirtIODevice *vdev, void *va)``

**va** - virtual address inside previously allocated memory block

Return value:

in case of error: NULL address

in case of success: physical address of the provided virtual address

**IRQL: <= DISPATCH**

## Freeing of previously allocated memory block(s)

``void VirtIOWdfDeviceFreeDmaMemory(VirtIODevice *vdev, void *va)``

``void VirtIOWdfDeviceFreeDmaMemoryByTag(VirtIODevice *vdev, ULONG groupTag)``

**va** - virtual address previously returned by VirtIOWdfDeviceAllocDmaMemory

**groupTag** - non-zero tag provided previously to VirtIOWdfDeviceAllocDmaMemory

**IRQL: PASSIVE**

## Allocation of sliced memory block

Use this method of allocation when you need many small chunks of physical memory of the same size

``PVIRTIO_DMA_MEMORY_SLICED VirtIOWdfDeviceAllocDmaMemorySliced(VirtIODevice *vdev, size_t blockSize, ULONG sliceSize)``

This procedure allocates block of one or more 4K pages from which you can allocate and free small memory slices.

It returns a pointer to a library-allocated structure VIRTIO_DMA_MEMORY_SLICED

IRQL: PASSIVE

structure VIRTIO_DMA_MEMORY_SLICED includes 3 methods:
* PVOID (*get_slice)(PVIRTIO_DMA_MEMORY_SLICED, PHYSICAL_ADDRESS *ppa)

The procedure allocates a slice from the block, fills the physical address of the slice and returns its virtual address

**IRQL: >= DISPATCH**

* void  (*return_slice)(PVIRTIO_DMA_MEMORY_SLICED, PVOID va)

This procedure returns the previosly allocated slice back to the block

**IRQL: >= DISPATCH**

* void  (*destroy)(PVIRTIO_DMA_MEMORY_SLICED)

The procedure frees the entire allocated block.

**IRQL: PASSIVE**

## Using driver-owned virtual memory buffers for DMA (guest-to-device direction only)

The library allows to convert any driver-owned memory buffer to scatter-gather table of physical fragments to be used for guest-to-device DMA operation.

The API for that is asynchronous because the WDF DMA API is also asynchronous, mapping of the buffer is executed by HAL on DISPATCH and requires the callback that receives the final mapping of the buffer. In order to avoid any misunderstanding with the lifetime of the buffer, the library reallocates the buffer and copies the content of the original buffer to the reallocated one.

To initiate the DMA operation the driver uses following sequence of operations
1. Define procedure that will called to provide scatter-gather table with following prototype:

``BOOLEAN (*VirtIOWdfDmaTransactionCallback)(PVIRTIO_DMA_TRANSACTION_PARAMS Parameters)``

2. Define locally and fill VIRTIO_DMA_TRANSACTION_PARAMS structure:
* **_(IN part, the driver fills)_**
* PVOID param1; /* scratch field to be used by the callback */
* PVOID param2; /* scratch field to be used by the callback */
* WDFREQUEST req; /* NULL */
* PVOID buffer;   /* buffer with data to be sent */
* ULONG size;     /* amount of data to be copied from buffer */
* ULONG allocationTag; /* used by the library for buffer reallocation */
* **_(the library fills when calls the callback procedure)_**
* WDFDMATRANSACTION transaction;
* PSCATTER_GATHER_LIST sgList;

3. Call the library procedure
``BOOLEAN VirtIOWdfDeviceDmaTxAsync(VirtIODevice *vdev, PVIRTIO_DMA_TRANSACTION_PARAMS params, VirtIOWdfDmaTransactionCallback callback)``

4. The library procedure reallocates and copies the content of the buffer and the VIRTIO_DMA_TRANSACTION_PARAMS structure, creates WDF DMA transaction object and starts it's execution. The WDF framework requests the HAL to map the reallocated buffer and provide the resulting scatther-gather list to the callback procedure.
5. If the buffer was mapped successfully, the driver callback procedure is called on IRQL=DISPATCH. It is expected to program device DMA with the provided scatter-gather table
6. If the callback procedure fails to use the scatter-gather table (I hardly can expect other reason than too many fragments) it should:
* call ``VirtIOWdfDeviceDmaTxComplete(virtio_device, Parameters->transaction)``
* return false
7. If the callback procedure successfully populates the physical addresses provided in the scatter-gather table, it should:
* save Parameters->transaction field in the data structure associated with the virtio queue entry
* return true
8. When the virtqueue entry is consumed by the device, call ``VirtIOWdfDeviceDmaTxComplete(virtio_device, transaction)``
* the library finishes the transaction and the scatter-gather table mapping finishes it's lifecycle

## Using memory buffers of WDF write request for guest-to-device DMA

The sequence of the operations is similar to the previous one (when the driver-owned buffer is used):
1. The driver defines the callback procedure

``BOOLEAN (*VirtIOWdfDmaTransactionCallback)(PVIRTIO_DMA_TRANSACTION_PARAMS Parameters)``
2. The driver defines lokally and fills VIRTIO_DMA_TRANSACTION_PARAMS structure:
* WDFREQUEST req = WDF request
* PVOID buffer = NULL
* ULONG size = 0  /* ignored */
* ULONG allocationTag = 0; /* ignored */
3. **The driver must leave the WDF write request uncancellable**
4. The driver calls

``BOOLEAN VirtIOWdfDeviceDmaTxAsync(VirtIODevice *vdev, PVIRTIO_DMA_TRANSACTION_PARAMS params, VirtIOWdfDmaTransactionCallback callback)``

5. The library request to map the buffer associated with the request and on HAL callback calls the driver callback on IRQL=DISPATCH
6. If the driver fails to populate the scatter-gather table into virtqueue entry, it:
* can complete the WDF request with failure status
* call ``VirtIOWdfDeviceDmaTxComplete(virtio_device, Parameters->transaction)``
* return false
7. If the driver succeeds to populate the virtqueue entry, it should:
* saves the Parameters->transaction field and WDF request handle in the data structure associated with the entry
8. When the virtqueue entry is consumed by the device, the driver should:
* call ``VirtIOWdfDeviceDmaTxComplete(virtio_device, Parameters->transaction)``
* complete the WDF request with successful status

**Important note:**

If the WDF request is cancellable, the driver has a problematic flow in case of request's cancellation. There is no possibility to cancel the virtqueue entry after it is pushed to the virtqueue. The driver should call ``VirtIOWdfDeviceDmaTxComplete(virtio_device, transaction)`` some time. It is possible that the device will consume data from physical address that is not valid anymore.

## Open issues and points to improve

* The library configures the WDF DMA enabler the same way for all the devices: maximum transfer size = 256 MB, unlimited number of fragments. Most of devices use short buffers for communication. But some of devices might be able to use large buffers when the number of fragments in fact might be limited. Need to verify that the API and the driver function corrrectly with large buffers.