# Porting Mvisor VGPU Driver to QEMU/Proxmox

## Key Finding
The Mvisor driver uses the **standard virtio-gpu protocol** (same command types, structures, and virtqueue layout as QEMU). The main differences are:

1. **Device ID**: Mvisor uses `1AF4:105B`, QEMU uses `1AF4:1050` — INF change only
2. **Virtqueue layout**: Mvisor uses COMMAND_QUEUE(0) + CONTROL_QUEUE(1). QEMU's virtio-gpu uses controlq(0) + cursorq(1). Need to verify queue mapping.
3. **Blob support**: Mvisor has blob resource support. QEMU's virtio-gpu-gl may or may not support all blob operations.
4. **Memory mapping**: `resource_map_blob` response includes GPA + size. QEMU's implementation may differ.

## Files to Modify

### kernelmode/vgpu/vgpu.inf
- Change `DEV_105B` → `DEV_1050` (already tested, works)

### kernelmode/vgpu/vgpu.c
- Review device initialization — may need adjustments for QEMU's feature negotiation
- Check virtqueue setup matches QEMU's virtio-gpu queue layout

### kernelmode/vgpu/command.c
- Queue indices: verify COMMAND_QUEUE=0 and CONTROL_QUEUE=1 match QEMU
- QEMU virtio-gpu uses: controlq=0, cursorq=1 — may need to swap

### kernelmode/vgpu/control.c
- Review capset negotiation — QEMU may report different capabilities
- Check blob resource handling compatibility

### kernelmode/vgpu/memory.c
- Review memory mapping — GPA handling may differ between Mvisor and QEMU

## Build Requirements
- Visual Studio 2019
- Windows Driver Kit (WDK) 10.0
- For usermode: VS2019 or MinGW-W64

## Test Plan
1. Change INF device ID ✓ (done, driver installs)
2. Fix virtqueue mapping if needed
3. Test basic capset negotiation
4. Test 3D context creation
5. Test OpenGL rendering via usermode ICD
6. Test with Vectra

## Current Status (2026-06-20)
- Fork created at: https://github.com/serenity-its-development/virtio-gpu-win-3d-driver
- INF modified for QEMU device ID — driver installs and binds
- Kernel driver loads but shows "Error" — likely virtqueue mismatch
- Usermode ICD (MvisorVGPUx64.dll + opengl32.dll) installed in System32
- OpenGL registry entries configured
- Need to debug kernel driver initialization failure
