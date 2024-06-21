＃ rust_shyper 试验与分析

### 生成rust_shyper可启动内核

编译rust shyper代码

`cargo build -Z build-std=core,alloc -Zbuild-std-features=compiler-builtins-mem --target /home/os/hypervisor/rust_shyper/cfg/aarch64.json --no-default-features --features "qemu," --release`

链接rust shyper和VM0 MVM Linux镜像文件：

`aarch64-none-elf-ld target/aarch64/release/librust_shyper.a -T linkers/aarch64.ld --defsym TEXT_START=0x40080000 -o target/aarch64/release/rust_shyper`

把Linux 4.9 （文件image/Image_vanilla）通过linkers/aarch64.ld包在.data段，位于地址_binary_vm0img_start；

`aarch64-none-elf-objcopy target/aarch64/release/rust_shyper -O binary target/aarch64/release/rust_shyper.bin`


### Qemu模拟器启动参数：
```
qemu-system-aarch64 -machine virt,virtualization=on,gic-version=2 -m 8g -cpu cortex-a57 -smp 4 -display none -global virtio-mmio.force-legacy=false -kernel target/aarch64/release/rust_shyper.bin -serial mon:stdio  -netdev user,id=n0,hostfwd=tcp::5555-:22 -device virtio-net-device,bus=virtio-mmio-bus.24,netdev=n0 -drive file=vm0.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.25
``` 

#### rust shyper执行入口：
src/arch/aarch64/start.rs::_start
