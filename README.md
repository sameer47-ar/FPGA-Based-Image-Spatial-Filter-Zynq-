# FPGA-Based Real-Time Image Spatial Filtering Accelerator

Hardware accelerator for 2D spatial image filtering using convolution, implemented on Xilinx Zynq platform.

This project implements a complete custom IP core for real-time neighborhood (convolution) processing of grayscale images. It supports 3×3 kernels for operations such as blurring, edge detection, and sharpening.

---

## Features

- Custom **line buffer architecture** with 4 rotating buffers
- Parallel data loading and convolution for improved throughput
- Pipelined **Multiply-Accumulate (MAC)** convolution engine
- Zero-latency prefetch design
- AXI-Stream interfaces for easy integration with DMA
- Interrupt-driven line-by-line processing
- Fully verified against MATLAB reference results (Lena image)

### Key Design Decisions

- **4 Line Buffers**: Allows continuous processing. While convolution runs on 3 lines, the 4th line is filled in parallel → ~50% better performance than a 3-buffer design.
- **Zero-latency read**: Combinational output + prefetch from line buffers.
- **Pipelined MAC**: Multiplication → Summation → Division by 9 (or kernel sum), with matching valid pipeline.
- **AXI-Stream + Interrupt**: Compatible with Zynq PS + DMA flow.

---

## Modules

| Module          | Description                                      |
|-----------------|--------------------------------------------------|
| `lineBuffer`    | Dual-port style RAM storing one image line (512 pixels). Supports 8-bit write and 24-bit (3-pixel) read. |
| `imageControl`  | Instantiates 4 line buffers, manages write demux, read mux, and FSM for continuous streaming. |
| `conv`          | 3×3 convolution engine using MAC operations. Currently configured for averaging (blur) filter. |
| `imageProcess`  | Top-level IP with AXI-Stream slave/master interfaces + interrupt. Includes output FIFO for back-pressure handling. |

---

## How to Simulate

1. Open the project in Vivado.
2. Add `tb.v` as a Simulation Source.
3. Place `lena.bmp` (512×512 grayscale) in the simulation directory.
4. Run Behavioral Simulation.
5. The testbench:
   - Reads the BMP header and copies it to `blurred.bmp`
   - Streams pixel data line-by-line using the interrupt handshake
   - Writes the processed pixels back to `blurred.bmp`
6. Open `blurred.bmp` to visually verify the blur effect (matches MATLAB `conv2` result).

---


---

## Future Improvements

- Parameterize image width/height and kernel size
- Support multiple kernels (Sobel, Laplacian, Gaussian, etc.) at runtime
- Add AXI-Lite control interface for kernel loading and status registers
- Color image (RGB) support


---

## License

This project is for educational purposes.
