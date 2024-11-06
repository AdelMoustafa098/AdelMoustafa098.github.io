---
title: "Image Processing Web App"
layout: post
date: 2024-06-10 2:51
image: https://www.simplilearn.com/ice9/free_resources_article_thumb/what_is_image_Processing.jpg
headerImage: True
projects: true
tag:
- markdown
- elements
star: true
category: blog
author: Adel Moussa
description: Image Processing Web App Using Python and Streamlit
---
<h1 style="text-align: center;">Image Processing</h1>

<p align="center">
  <img src="https://github.com/AdelMoustafa098/AdelMoustafa098/assets/43845826/c758413a-cf6a-4bae-ab26-a654f4cf4d1c" />
</p>

## Introduction

In today's digital era, the power of visual data is undeniable. From enhancing photos to detecting objects, 
image processing has revolutionized the way we interpret and manipulate images. Image processing involves 
a series of operations that convert an image into a more useful form. It encompasses various tasks such as filtering, enhancing, and analyzing images to extract meaningful information.

## Implemented Methods

To demonstrate these concepts, I created an **Image Web Processing** App that performs various operations on images to enhance their quality and make them more informative. Each function in this app serves a unique purpose in transforming raw images into refined, valuable visual data. Let’s dive into the methods implemented in this app and explore the logic and theory behind each one."


1. `get_image`

    The get_image method simply returns the current state of the image. It’s used to retrieve the image in its current processed or unprocessed state as a Numpy array, allowing for further operations or display.

    ```python
    def get_image(self):
        return self.image
    ```
2. `reset_image`  

    The reset_image method restores the image to its original, unaltered state. This functionality is crucial in applications where multiple transformations might be applied, as it allows you to revert any changes and start from the original image without reloading.

    ```python
    def reset_image(self):
        self.image = self.original_image
    ```
3. `convert_to_gray`   

    Converting an image to grayscale simplifies it by removing color information, reducing each pixel to a single intensity value. Grayscale images are commonly used in applications like edge detection and object recognition, where color isn’t necessary. The method calculates the grayscale intensity by applying specific weights to each color channel (Red, Green, and Blue) based on human perception of brightness.

    ```python
    def convert_to_gray(self, image):
        b, g, r = image[:, :, 0], image[:, :, 1], image[:, :, 2]
        gray = 0.2989 * r + 0.5870 * g + 0.1140 * b
        self.image = gray.astype(np.uint8)
        return gray.astype(np.uint8)
    ```
4. `add_salt_pepper_noise`  

    "Salt and pepper" noise simulates real-world noise by adding random black and white pixels to an image, which can imitate interference in old cameras or compression artifacts. This type of noise is useful for testing the robustness of filtering methods since it's challenging to remove without affecting image quality.

    ```python
    def add_salt_pepper_noise(self):
      # make sure image is gray
      if not self.is_gray(self.image):
          self.image = self.convert_to_gray(self.image)

      row, col = self.image.shape
      selected_pixel = np.random.randint(100, 5000)

      # Generate coordinates for white (salt) noise
      white_coords = (
          np.random.randint(0, row, selected_pixel),
          np.random.randint(0, col, selected_pixel),
      )
      self.image[white_coords] = 255

      # Generate coordinates for black (pepper) noise
      black_coords = (
          np.random.randint(0, row, selected_pixel),
          np.random.randint(0, col, selected_pixel),
      )
      self.image[black_coords] = 0
    ```
5. `add_gaussian_noise`  

    Gaussian noise is characterized by a bell curve distribution, which means most noise values are close to the mean with a few high and low extremes. This method is often used to test filters designed to reduce noise while preserving important details. Gaussian noise is common in low-light photography or high-ISO settings in cameras.

    ```python
    def add_gaussian_noise(self):
       # make sure image is gray
        if not self.is_gray(self.image):
            self.image = self.convert_to_gray(self.image)

        row, col = self.image.shape
        # create gaussian noise
        mean = 0.0
        std = 15.0
        noise = np.random.normal(mean, std, size=(row, col))

        # apply noise
        self.image = np.add(self.get_image(), noise)
        self.image = self.image.astype(np.uint8)
    ```

6. `add_uniform_noise`

    Unlike Gaussian noise, which has a bell curve distribution, uniform noise assigns equal probability to all intensity values in a certain range. This type of noise is useful for simulations in specific applications where all intensities are equally likely to be affected, such as in synthetic image generation.  

    ```python
    def add_uniform_noise(self):
      # make sure image is gray
        if not self.is_gray(self.image):
            self.image = self.convert_to_gray(self.image)

        # create uniform image
        row, col = self.image.shape
        noise = np.random.uniform(-20, 20, size=(row, col))

        # apply noise
        self.image = np.add(self.image, noise)
        self.image = self.image.astype(np.uint8)
    ```

7. `apply_mask`

    Masks are fundamental in image processing and can be used for blurring, sharpening, edge detection, and more. The apply_mask method applies a convolution between the image and a given mask (kernel), which defines how each pixel’s neighborhood contributes to the new value. This is essential in tasks where we want to extract or emphasize specific features.

    ```python
    def apply_mask(self, image: np.array, mask: np.array):
      if not self.is_gray(image):
          image = self.convert_to_gray(image)
      image = signal.convolve2d(image, mask)
      masked_image = image.astype(np.uint8)
      return masked_image
    ```

8. `avg_filter`

    An average filter, or mean filter, reduces noise by averaging pixel values within a defined window. It’s useful for reducing random noise, though it may slightly blur the image. This filter is implemented here as a 3x3 mask, meaning each pixel is replaced by the average of itself and its 8 surrounding pixels.

    ```python
    def avg_filter(self):
        mask = np.ones([3, 3], dtype=int)
        mask = mask / 9
        self.image = self.apply_mask(self.image, mask)
    ```
9. `gaussian_filter`

   A Gaussian filter applies a Gaussian kernel, which smooths images while preserving edges better than an average filter. It’s particularly effective at reducing Gaussian noise, where intensity values vary randomly. Here, a 9x9 kernel with a sigma of 2.6 is used, striking a balance between smoothness and detail preservation.

   ```python
   def gaussian_filter(self):
        # Define the standard deviation (sigma) and the size of the Gaussian kernel
        sigma = 2.6
        kernel_size = (9, 9)

        # Calculate the center coordinates of the kernel
        center_y, center_x = [(size - 1.0) / 2.0 for size in kernel_size]

        # Generate a grid of (y, x) coordinates
        y, x = np.ogrid[-center_y : center_y + 1, -center_x : center_x + 1]

        # Compute the Gaussian function
        gaussian_kernel = np.exp(-(x * x + y * y) / (2.0 * sigma * sigma))

        # Set very small values to zero
        gaussian_kernel[
            gaussian_kernel
            < np.finfo(gaussian_kernel.dtype).eps * gaussian_kernel.max()
        ] = 0

        # Normalize the kernel so that its sum is 1
        kernel_sum = gaussian_kernel.sum()
        if kernel_sum != 0:
            gaussian_kernel /= kernel_sum

        # Apply the Gaussian filter to the image
        self.image = self.apply_mask(self.image, gaussian_kernel)
   ``` 

10. `sobel_edge`

    The Sobel edge detection method highlights edges by calculating the intensity gradient in two perpendicular directions. Sobel filters emphasize areas of significant intensity change, making it useful for applications like object boundary detection. This method applies separate filters for the x and y directions, then combines the results to emphasize edges in all orientations.

    ```python
    def sobel_edge(self):
        # Apply Gaussian filter to smooth the image
        self.gaussian_filter()

        # Define Sobel kernels
        kx = np.array([[1, 0, -1], [2, 0, -2], [1, 0, -1]])
        ky = kx.T

        # Compute gradients using convolution
        Ix = signal.convolve2d(self.get_image(), kx, mode="same", boundary="symm")
        Iy = signal.convolve2d(self.get_image(), ky, mode="same", boundary="symm")

        # Calculate magnitude and direction of the gradient
        magnitude = np.hypot(Ix, Iy)
        direction = np.arctan2(Iy, Ix)

        # Convert direction to degrees and shift range from [-180, 180] to [0, 360]
        direction = np.rad2deg(direction) + 180

        # Convert to uint8 type
        self.image = np.clip(magnitude, 0, 255).astype(np.uint8)
    ```
11. `roberts_edge`

    The Roberts edge detection operator uses two diagonal convolution kernels, which are particularly sensitive to diagonal edges. While it doesn’t handle noisy images as well as Sobel, it’s more effective for finding sharp, simple edges. This method is often used in applications that prioritize fast computation over fine detail accuracy.

    ```python
    def roberts_edge(self):
        mask_x = np.array([[0, 0, 0], [0, 1, 0], [0, 0, -1]])

        mask_y = np.array([[0, 0, 0], [0, 0, 1], [0, -1, 0]])

        self.gaussian_filter()
        mask_x_dirc = self.apply_mask(self.image, mask_x)
        mask_y_dirc = self.apply_mask(self.image, mask_y)

        gradient = np.sqrt(np.square(mask_x_dirc) + np.square(mask_y_dirc))
        self.image = np.uint8((gradient * 255.0) / gradient.max())

    ```

## Closing Remarks 

With these methods, the Image Web Processing App can transform images through various filters, noise additions, and edge detections, making it a powerful tool for refining visual data for analysis or presentation. Each technique has a unique role, from enhancing image clarity to isolating critical features, building a solid foundation for advanced image processing.

if you want to take a look on the whole project you can do so by visiting my GitHub profile via this [link](https://github.com/AdelMoustafa098/Image_Processing_web_app)