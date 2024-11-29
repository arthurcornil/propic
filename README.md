# ProPic - Quick Profile Picture Creator

ProPic is a lightweight web app that allows users to create a stylish profile picture with just a few clicks! Users can upload a photo, remove its background (processed directly in the browser), and customize their image with a colourful circular background. The app is built using Svelte and processes everything on the client side, ensuring privacy.

## Features

- **Background Removal:** Removes the background from your image, all within the browser.
- **Customizable Colors:** Add a colorful circle behind your silhouette.
- **Image Adjustment:** Fine-tune the zoom and position of your image for the perfect fit.
- **Downloadable Output:** Download your customized profile picture in PNG format.
- **Privacy First:** All processing is done locally in your browser—no uploads to a server.

## Tech Stack

- **Framework:** [Svelte](https://svelte.dev)
- **Styling:** [Tailwind CSS](https://tailwindcss.com)
- **UI Components:** [shadcn-svelte](https://www.shadcn-svelte.com/#)
- **Background Removal:** [imgly/background-removal](https://github.com/imgly/background-removal)
- **Cropping Utility:** [autocrop-js](https://github.com/freearhey/autocrop-js)
- **Canvas Rendering:** [html2canvas](https://html2canvas.hertzen.com)

## How to Run Locally

1. Clone this repository:
    ```bash
    git clone git@github.com:arthurcornil/propic.git
    cd propic
    ```

2. Install dependencies:
    ```bash
    pnpm install
    ```

3. Start the development server:
    ```bash
    pnpm run dev
    ```
## How to Use

1. **Upload an Image:** Drag and drop your image onto the upload area or click to select a file.
2. **Background Removal:** Wait for the app to process and remove the background.
3. **Customize:** 
   - Choose a background color.
   - Adjust the image's position and zoom using sliders.
4. **Download:** Click the "Download my ProPic" button to save your profile picture.

## Contributing

Contributions are welcome! If you have ideas for improving the app or want to fix a bug, feel free to fork the repository and submit a pull request.

## License

This project is licensed under the MIT License. See the `LICENSE` file for more details.

---

Enjoy creating your profile pictures with **ProPic**! 🎉

