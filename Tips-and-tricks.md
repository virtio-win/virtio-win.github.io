## Guest (Linux/Windows) <--> Host (Linux) stdout redirection

Let's start with the console app. There is an existing mechanism of `virtio-serial` ports 
and consoles over virtio serial. 

On the host side, you can bind `virtio-serial` port to `chardev` that can even be stdout.
Examples:
 - https://stackoverflow.com/questions/68277557/qemu-virtio-virtconsole-devices-explained
 - https://fedoraproject.org/wiki/Features/VirtioSerial

On the Windows guest side, you will need to write some app that redirects the console to the virtio-serial port. 
Examples of the port usage can be found in qemu-ga (guest agent): https://github.com/qemu/qemu/blob/master/qga/channel-win32.c#L283

On the Linux guest side, console ports will be available as `/dev/hvc*` or `/dev/vpor*` devices depending on the command line.

Regarding config from host to guest. It can be using the same or another virtio-serial port as well.
