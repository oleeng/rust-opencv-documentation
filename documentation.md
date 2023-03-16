# Documentaion
This documentaion lists all Rust/OpenCV functions I used so far and tries to eplain them better than the official documentaion does. Hopefully this makes future work easier.

## imread
This function opens an image
```rust
pub fn imread(filename: &str, flags: i32) -> Result<Mat>
```

### Parameters:
- filename: String of the path to the image
- flags: Flag for how the image should be read
    - IMREAD_UNCHANGED
        - this is the defauld mode
        - Reads the image as it is including the alpha channel. The output type is CV_8UC4 if the image has an alpha channel and CV_8UC3 otherwise
    - IMREAD_GRAYSCALE
        - Reads the image as a grayscale image. The output type is CV_8UC1
    - IMREAD_COLOR
        - Reads the image as a color image. The output type is CV_8UC3

### Usage
```rust
use opencv::imgcodecs;
let img = imgcodecs::imread("path/to/image.png", imgcodecs::IMREAD_GRAYSCALE).unwrap();
```

## threshold
This function applies a threshold function to the image
```rust
pub fn threshold(src: &dyn ToInputArray, dst: &mut dyn ToOutputArray, thresh: f64, maxval: f64, typ: i32) -> Result<f64>
```

### Parameters
- src: input array of the image
- dst: output array of the result image
- thresh: threshold value
- maxval: maximum value to use in the threshold function
- typ: flag for the type of threshold function
    - THRESH_BINARY
        - If the pixel value is larger than the threshold value set it to the maximum value otherwise to 0
    - THRESH_BINARY_INV
        - If the pixel value is lager than the threshold value set it to 0 otherwise to the maximum value
    - some more

### Usage
```rust
use opencv::imgproc;
let mut outImg = Mat::default();
imgproc::threshold(&img, &mut outImg, 170:0, 255.0, imgproc::THRESH_BINARY);
```

## imshow
This function displays an image in a window

### Usage
```rust
use opencv::highgui;
highgui::named_window("window", highgui::WINDOW_FULLSCREEN).unwrap();
highgui::imshow("window", &img).unwrap();
let key = highgui::wait_key(0).unwrap();
```
For the named window function there are also other flags defining the behaviour available
- WINDOW_NORMAL
    - The window can be resized by the user. This is the default behavior.
- WINDOW_AUTOSIZE
    - The window size is automatically adjusted to fit the image displayed in it
- WINDOW_FULLSCREEN
    - The window will be created in full screen mode
- WINDOW_FREERATIO
    - The aspect ratio of the window can be changed freely by the user
- WINDOW_KEEPRATIO
    - The aspect ratio of the window is fixed

## bitwise_xor
This function applies a bitwise XOR to the two input images. The function for bitwise or and and work identical. Just change the operator in the function name
```rust
pub fn bitwise_xor(src1: &dyn ToInputArray, src2: &dyn ToInputArray, dst: &mut dyn ToOutputArray, mask: &dyn ToInputArray) -> Result<()>
```
### Parameters
- src1: First imput image
- src2: Second input Image
- dst: mutatable image to save the result to
- mask: mask image to select the regions the operation should be applied to

### Usage
```rust
use opencv::core;
let mut result = Mat::default();
core::bitwise_xor(&img1, &img2, &mut result, &Mat::default()).unwrap();
```

## morphology_ex
This function applies a morphological operation like EROSION, DIALATION, OPEN, CLOSE to an image
```rust
pub fn morphology_ex(src: &dyn ToInputArray, dst: &mut dyn ToOutputArray, op: i32, kernel: &dyn ToInputArray, anchor: Point, iterations: i32, border_type: i32, border_value: Scalar) -> Result<()>
```

### Parameters
- src: Input image
- dst: Output Image
- op: flag for the used operation
    - MORPH_ERODE
    - MORPH_DILATE
    - MORPH_OPEN
    - MORPH_CLOSE
- kernel: The structuring element defining the shape of the used neighborhood. Can be of type Mat or created using get_structuring_element
    - ```rust
      use opencv::core::*
      let kernel = Mat::ones(5, 5, CV_8U).unwrap();
      ```
    - the get_structuring_element has three parameters. The used type, the size of the kernel and the anchor point of the kernel. (-1, -1) correspods to the center of the created kernel.
        - ```rust
          use opencv::imgproc;
          let kernel = imgproc::get_structuring_element(
              imgproc::MORPH_RECT,
              Size::new(5, 5),
              Point::new(-1, -1),
          ).unwrap();
          ```
        - ```rust
          use opencv::imgproc;
          let kernel = imgproc::get_structuring_element(
              imgproc::MORPH_CROSS,
              Size::new(5, 5),
              Point::new(-1, -1),
          ).unwrap();
          ```
        - ```rust
          use opencv::imgproc;
          let kernel = imgproc::get_structuring_element(
              imgproc::MORPH_ELLIPSE,
              Size::new(5, 5),
              Point::new(-1, -1),
          ).unwrap();
          ```
- anchor:  The anchor point of the used kernel. (-1, -1) correspods to the center of the created kernel
- iterations: Number of iterations the operation gets applied
- border_type: type for how the border pixels should be handled
    - BORDER_CONSTANT
    - BORDER_REPLICATE
    - BORDER_REFLECT
    - BORDER_WRAP
- border_value: The pixel value that should be used. Only used if border type is BORDER_CONSTANT

### Usage
```rust
use opencv::imgproc;
use opencv::core::*
let kernel = imgproc::get_structuring_element(
    opencv::imgproc::MORPH_RECT,
    Size::new(3, 3),
    Point::new(1, 1)
).unwrap();
let mut result = Mat::default();
imgproc::morphology_ex(&img, &mut result, imgproc::MORPH_OPEN, &kernel, Point::new(-1, -1), 1, BORDER_CONSTANT, Scalar::all(0.0)).unwrap();
```

## hough_lines_p
Finds lines in a binary black and white image using the hough transformation. 
```rust
pub fn hough_lines_p(image: &dyn ToInputArray, lines: &mut dyn ToOutputArray, rho: f64, theta: f64, threshold: i32, min_line_length: f64, max_line_gap: f64) -> Result<()>
```

### Parameters
- image: Input image
- lines: output array for the found lines. A line is represented by four values: (x1, y1, x2, y2) where (x1, y1) and (x2, y2) are the endpoints of the found line
- rho: Distance resolution of the accumulator in pixels
- theta: Angle resolution of the accumulator in radians
- threshold: Accumulator threshold parameter. Only those lines are returned that get enough votes
- min_line_length:  Minimum line length
- max_line_gap: Maximum allowed gap between points on the same line to link them

### Usage
```rust
use opencv::imgproc;
use opencv::core::Point;
let mut lines = Mat::default();
imgproc::hough_lines_p(&img, &mut lines, 1.0, std::f64::consts::PI / 180.0, 500, 10.0, 200.0,).unwrap();
for i in 0..lines.cols().unwrap() {
    let line = lines.at_row::<i32>(0).unwrap().at(i).unwrap();
    let p1 = Point::new(line[0], line[1]);
    let p2 = Point::new(line[2], line[3]);
}
```
or
```rust
use opencv::imgproc;
use opencv::core::Point;
let mut lines = opencv::types::VectorOfVec4i::new();
imgproc::hough_lines_p(&img, &mut lines, 1.0, std::f64::consts::PI / 180.0, 500, 10.0, 200.0,).unwrap();
for line in lines.iter() {
    let p1 = Point::new(line[0], line[1]);
    let p2 = Point::new(line[2], line[3]);
}
```