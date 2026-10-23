# Image Stitching on Xilinx PYNQ-Z2

Hardware/Software Co-design coursework, Lakehead University (November 2023). An OpenCV image stitching pipeline adapted to run on the Xilinx PYNQ-Z2 board, within the board's resolution and memory constraints.

# Overview

Image stitching combines multiple photographs with overlapping fields of view into a single panoramic composite. This project implements stitching with OpenCV's Stitcher module, wrapped in a pipeline built for an embedded target: inputs are resolution-optimized to fit the PYNQ-Z2's constraints before stitching, stitching failures are retried, and the result is presented with the input images above the stitched panorama, as specified in the assignment.

The deliverables are preserved as submitted: the executed Jupyter notebook, the assignment report, the three input photographs, and the run screenshots.

# Problem Statement

Produce a stitched panorama from three input images on the Xilinx PYNQ-Z2, an embedded platform with limited memory and processing headroom compared to a desktop, such that oversized inputs do not crash the run and transient stitching failures are handled rather than fatal.

# Objectives

1. Implement image stitching using OpenCV's Stitcher module in Python.
2. Adapt the pipeline to the PYNQ-Z2: optimize input resolution against the board's maximum constraints while preserving aspect ratio.
3. Make the run robust: retry failed stitches a bounded number of times before reporting failure.
4. Display inputs and the stitched result in the specified layout and save the output image.

# Hardware Setup

- Xilinx PYNQ-Z2 development board (Zynq-7000 SoC).
- The solution runs in Python on the board's ARM processing system under the PYNQ framework, in a Jupyter notebook. No custom FPGA fabric (PL) design or overlay is included; the board is the execution target and its constraints drive the resolution optimization, which is the hardware-aware part of the work.

# Software Setup

- Python with OpenCV (`cv2`), NumPy, Matplotlib.
- `image_stitching_pynq.ipynb`: the complete notebook (load, optimize, stitch, display, save).
- Inputs: `assets/img11.jpg`, `assets/img22.jpg`, `assets/img33.jpg` (original file names preserved).
- Full write-up: `Nahiyan_Al_Mahmood_Assignment2_Report.pdf`.

# Architecture

```text
Input images (3)
      ↓
Resolution optimization (get_optimized_resolution: fit board max constraints, keep aspect ratio)
      ↓
Pairwise stitching (stitch_images: OpenCV Stitcher, bounded retries)
      ↓
Display (inputs in a row, stitched panorama below) + save stitched_image.jpg
```

# Methodology

- `get_optimized_resolution` computes each image's aspect ratio and scales it down to the maximum width/height the board target allows.
- `stitch_images` stitches consecutive image pairs with OpenCV's Stitcher, feeding each successful result into the next step. On failure it retries a specified number of times before giving up, because the Stitcher can fail transiently on low-overlap pairs.
- The notebook displays the three inputs, then the stitched panorama, and writes the result to `stitched_image.jpg`.

# Test Matrix

| Case | Expected | Observed (per report and screenshots) |
|---|---|---|
| Three provided inputs, standard run | Single stitched panorama saved and displayed | Stitching completed successfully; cohesive panorama produced |
| Oversized input images | Run within board limits after optimization | Unoptimized oversized files fail to execute on the board; resolution optimization addresses this |
| Low-overlap pair | Bounded retries, then a reported failure rather than a crash | Retry mechanism introduced for this case |

# Investigation Process

Two failure modes drove the design, both documented in the report's "Challenges Faced and Mitigation" section: board memory limits rejecting large inputs (mitigated by resolution optimization), and intermittent Stitcher failures on limited-overlap images (mitigated by retries). Algorithm selection (OpenCV Stitcher versus a hand-rolled feature pipeline) was the third decision, balancing accuracy against real-time suitability on the board.

# Evidence

- `image_stitching_pynq.ipynb`: the executed notebook (renamed from the submitted `Nahiyan_AlMahmood_1155169_Asignment2.ipynb`).
- `Nahiyan_Al_Mahmood_Assignment2_Report.pdf`: the assignment report (introduction, methodology, results, challenges, conclusion).
- `assets/`: the three original input photographs and four run screenshots, including the stitched result.

# Results

The pipeline stitched the three inputs into a single panoramic image, displayed in the report (Fig. 2) and preserved in the run screenshots. The report's observation: the stitched image effectively combines the visual information from the inputs into a seamless panorama, and the resolution optimization keeps the run compatible with the board's constraints.

# Limitations

- Coursework scope: one assignment, a fixed set of three input images, no benchmark suite or timing measurements.
- Execution is on the ARM processing system only; the FPGA fabric is not used, so no hardware acceleration is claimed.
- OpenCV's Stitcher is a black box here; feature matching internals were not modified or analysed.

# Lessons Learned

- Embedded targets change the problem: input size is a correctness constraint, not a performance detail, when the board refuses oversized files outright.
- Defensive structure (bounded retries, explicit failure reporting) matters more on hardware you cannot attach a debugger to mid-run.

# Future Work

- Accelerate feature detection/matching in the FPGA fabric with a PYNQ overlay, and measure the difference.
- Extend from a fixed three-image set to arbitrary sequences, with timing and memory measurements on the board.

# Provenance

Coursework by Nahiyan Al Mahmood (Lakehead University, Hardware/Software Co-design, Assignment 2, November 2023). Files are the original submission artifacts, recovered from Google Drive in October 2026 and published with light renaming of the notebook only; contents are unedited.
