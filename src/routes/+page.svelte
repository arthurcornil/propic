<script lang="ts">
	import { removeBackground } from '@imgly/background-removal';
    import { UploadSolid } from 'flowbite-svelte-icons';
    import { fly } from 'svelte/transition';

    let file: File | null = $state(null);
    let loading = $state(false);
    let imageUrl: string = $state('');
    let isImageProcessed = $state(false);
    let color: string = $state('#679c67');
    let step: number = $state(0);

    async function createPop(event: DragEvent) {
        event.preventDefault();
        event.stopPropagation();

        const droppedFiles = event.dataTransfer?.files;

        if (!droppedFiles || droppedFiles.length === 0) {
            return ;
        }
        file = droppedFiles[0];
        const data = await removeBackgroundFromFile();
        const trimmedBlob = await trimTransparentEdges(data);
        imageUrl = URL.createObjectURL(trimmedBlob);
        isImageProcessed = true;
        loading = false;
    }

    async function removeBackgroundFromFile(): Promise<Blob> {
        if (!file) {
            throw Error('No file selected');
        }
        loading = true;
        try {
            const data: Blob = await removeBackground(file);
            loading = false;
            return data;
        } catch (error) {
            loading = false;
            throw Error(`Error: ${error}`);
        }
    }

    async function trimTransparentEdges(inputBlob: Blob): Promise<Blob> {
        return new Promise((resolve, reject) => {
            const reader = new FileReader();

            reader.onload = () => {
                const image = new Image();
                image.onload = () => {
                    const canvas = document.createElement("canvas");
                    const ctx = canvas.getContext("2d");

                    if (!ctx) {
                        reject(new Error("Could not create canvas context."));
                        return;
                    }

                    canvas.width = image.width;
                    canvas.height = image.height;

                    ctx.drawImage(image, 0, 0);

                    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
                    const { data, width, height } = imageData;

                    let top = 0, left = 0, right = width - 1, bottom = height - 1;

                    topLoop: for (let y = 0; y < height; y++) {
                        for (let x = 0; x < width; x++) {
                            const alpha = data[(y * width + x) * 4 + 3];
                            if (alpha > 0) {
                                top = y;
                                break topLoop;
                            }
                        }
                    }

                    bottomLoop: for (let y = height - 1; y >= 0; y--) {
                        for (let x = 0; x < width; x++) {
                            const alpha = data[(y * width + x) * 4 + 3];
                            if (alpha > 0) {
                                bottom = y;
                                break bottomLoop;
                            }
                        }
                    }

                    leftLoop: for (let x = 0; x < width; x++) {
                        for (let y = 0; y < height; y++) {
                            const alpha = data[(y * width + x) * 4 + 3];
                            if (alpha > 0) {
                                left = x;
                                break leftLoop;
                            }
                        }
                    }

                    rightLoop: for (let x = width - 1; x >= 0; x--) {
                        let hasNonTransparentPixel = false;
                        for (let y = 0; y < height; y++) {
                            const alpha = data[(y * width + x) * 4 + 3];
                            if (alpha > 0) {
                                hasNonTransparentPixel = true;
                                break;
                            }
                        }
                        if (hasNonTransparentPixel) {
                            right = x;
                            break rightLoop;
                        }
                    }

                    const trimmedWidth = right - left + 1;
                    const trimmedHeight = bottom - top + 1;

                    if (trimmedWidth <= 0 || trimmedHeight <= 0) {
                        reject(new Error("Image is fully transparent or invalid."));
                        return;
                    }

                    const trimmedCanvas = document.createElement("canvas");
                    const trimmedCtx = trimmedCanvas.getContext("2d");

                    if (!trimmedCtx) {
                        reject(new Error("Could not create trimmed canvas context."));
                        return;
                    }

                    trimmedCanvas.width = trimmedWidth;
                    trimmedCanvas.height = trimmedHeight;

                    trimmedCtx.drawImage(
                        canvas,
                        left,
                        top,
                        trimmedWidth,
                        trimmedHeight,
                        0,
                        0,
                        trimmedWidth,
                        trimmedHeight
                    );

                    trimmedCanvas.toBlob((trimmedBlob) => {
                        if (trimmedBlob) {
                            resolve(trimmedBlob);
                        } else {
                            reject(new Error("Failed to create Blob from trimmed canvas."));
                        }
                    }, "image/png");
                };

                image.onerror = () => {
                    reject(new Error("Failed to load image."));
                };

                image.src = reader.result as string;
            };

            reader.onerror = () => {
                reject(new Error("Failed to read Blob."));
            };

            reader.readAsDataURL(inputBlob);
        });
    }

    $effect(() => {
        if (loading) {
            step = 1;
            setTimeout(() => {
                step = 2;
            }, 5000);
	}}); 
</script>
<main class="p-4 flex items-center flex-col gap-12 pt-12">
    <div class="flex flex-col items-center gap-4">
        <h1 class="text-gray-900 text-3xl font-bold">PicoPop</h1>
        <h1 class="text-gray-700 text-xl font-semibold">Generate a cool profile picture in seconds!</h1>
    </div>
    <div>
        <div class="mb-12 flex flex-col items-center">
            {#if step === 1}
                <span class="text-gray-600 text-lg font-normal absolute" transition:fly={{ delay: 500, duration: 500 }}>
                    Hang tight, we're removing the background from your image.
                </span>
            {:else if step === 2}
                <span class="text-gray-600 text-lg font-normal absolute" transition:fly={{ delay: 500, duration: 500 }}>
                    It's all happenning live in your browser, so it might take some time ...
                </span>
            {:else if step === 0}
                <span class="text-gray-600 text-lg font-normal absolute" transition:fly={{ duration: 500 }}>
                    Drop your photo or click down below to import it.
                </span>
            {:else if isImageProcessed}
                <span class="text-gray-600 text-lg font-normal absolute" transition:fly={{ delay: 500, duration: 500 }}>
                    Edit your profile picture as you want!
                </span>
            {/if}
        </div>
        <div class="flex justify-center items-center">
            {#if !isImageProcessed && !loading}
                <div 
                    class="absolute rounded-full bg-gray-950 bg-opacity-50 border-2 border-gray-200 border-dashed w-56 h-56 flex justify-center items-center p-4 text-gray-100"
                    on:dragover={(event) => event.preventDefault()}
                    on:dragenter={(event) => event.preventDefault()}
                    on:drop={createPop}
                >
                    <UploadSolid class="h-24 w-24"/>
                </div>
            {/if}
            {#if loading}
                <div role="status" class="absolute">
                    <svg aria-hidden="true" class="w-8 h-8 text-gray-200 animate-spin dark:text-gray-600 fill-gray-600" viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
                        <path d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z" fill="currentColor"/>
                        <path d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z" fill="currentFill"/>
                    </svg>
                    <span class="sr-only">Loading...</span>
                </div>
            {/if}
            {#if isImageProcessed}
                <img src={imageUrl} class="max-w-60"/>
            {/if}
            <input type="file" on:change={createPop} hidden>
            <div class="rounded-full w-60 h-60" style="background-color: {color}"></div>
    </div>
    {#if isImageProcessed}
        <div class="grid grid-cols-3 gap-2">
            <div>
                <h1 class="text-gray-700 text-xl font-semibold mb-5">Choose a colour</h1>
                <input type="color" bind:value={color}>
            </div>
        </div>
    {/if}
</main>
