Here is the way to match the build number to source:

Look at the last number of the version (x.x.x.1000 in current build). The first part of it is a build number (in this case 100).
Now go to the stable branch of the source https://github.com/YanVugenfirer/kvm-guest-drivers-windows/tree/stable and look for the tags or commit message with message mmXXX <-->bYYY. YYY - will be the external build number (100): https://github.com/YanVugenfirer/kvm-guest-drivers-windows/commit/8a64980edc9f85dc0f1265827a23f2c053a4821f