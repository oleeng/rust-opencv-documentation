# Documentaion
This documentaion lists all Rust/OpenCV functions I used so far and tries to eplain them better than the official documentaion does. Hopefully this makes future work easier.

## imread
This function opens an image
```rust
pub fn imread(filename: &str, flags: i32) -> Result<Mat>
```
Parameters:
- filename: String of the path to the image
- flags: Flag for how the image should be read
    - IMREAD_UNCHANGED
        - this is the defauld mode
        - Reads the image as it is including the alpha channel. The output type is CV_8UC4 if the image has an alpha channel and CV_8UC3 otherwise
    - IMREAD_GRAYSCALE
        - Reads the image as a grayscale image. The output type is CV_8UC1
    - IMREAD_COLOR
        - Reads the image as a color image. The output type is CV_8UC3

Usage
```rust
use opencv::imgcodecs;
let img = imgcodecs::imread("path/to/image.png", imgcodecs::IMREAD_GRAYSCALE).unwrap();
```

## 