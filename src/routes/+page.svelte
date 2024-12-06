<script lang="ts">
    import { removeBackground } from '@imgly/background-removal';
    import { UploadSolid, GithubSolid } from 'flowbite-svelte-icons';
    import { fly } from 'svelte/transition';
    import autocrop from 'autocrop-js';
    import { Slider } from "$lib/components/ui/slider";
    import { Button } from "$lib/components/ui/button";
    import html2canvas from 'html2canvas';
    import { toast } from 'svelte-sonner';

    let file: File | null = $state(null);
    let loading = $state(false);
    let imageUrl: string = $state('');
    let isImageProcessed = $state(false);
    let color: string = $state('#679c67');
    let step: number = $state(0);
    let yImagePosition: number[] = $state([40]);
    let xImagePosition: number[] = $state([0]);
    let imageZoom: number[] = $state([1]);

    let proPic: HTMLElement;
    let fileInput: HTMLInputElement;

    async function processImage(event: Event | DragEvent) {
        event.preventDefault();
        event.stopPropagation();

        const droppedFiles = event instanceof DragEvent 
            ? event.dataTransfer?.files 
            : (event.target as HTMLInputElement).files;

        if (!droppedFiles || droppedFiles.length === 0) return;

        file = droppedFiles[0];
        const validTypes = ['image/png', 'image/jpeg'];

        if (!validTypes.includes(file.type)) {
            toast.error("Import a jpg or png image");
            return;
        }

        loading = true;
        const data: Blob = await removeBackgroundFromFile();
        const url: string = URL.createObjectURL(data);
        const result = await autocrop(url, { alphaThreshold: 10 });

        imageUrl = result.dataURL;
        loading = false;
        isImageProcessed = true;
        fileInput.value = '';
    }

    async function removeBackgroundFromFile(): Promise<Blob> {
        if (!file) throw Error('No file selected');

        try {
            return await removeBackground(file);
        } catch (error) {
            loading = false;
            throw Error(`Error: ${error}`);
        }
    }

    function triggerFileInput() {
        fileInput.click();
    }

    async function downloadProPic() {
        if (!isImageProcessed) return;

        const canvas = await html2canvas(proPic, { backgroundColor: null });
        const link = document.createElement('a');
        link.download = 'propic.png';
        link.href = canvas.toDataURL('image/png');
        link.click();
    }

    $effect(() => {
        if (loading) {
            step = 1;
            setTimeout(() => {
                step = 2;
            }, 5000);
        }
    }); 
</script>

<main class="p-4 flex items-center flex-col gap-12 pt-12 pb-12 relative min-h-[calc(100svh-4rem)]">
    <div class="flex flex-col items-center gap-4">
        <span>
            <img src="/icon.png" alt="" class="h-24 w-24">
            <h1 class="text-gray-900 text-3xl font-bold">ProPic</h1>
        </span>
        <h1 class="text-gray-700 text-xl font-semibold">Generate a cool profile picture in seconds!</h1>
    </div>
    <div class="mb-12 flex flex-col items-center">
        {#if step === 1}
            <span class="text-gray-600 text-lg font-normal absolute" transition:fly={{ delay: 700, duration: 500 }}>
                Hang tight, we're removing the background from your image.
            </span>
        {:else if step === 2 && !isImageProcessed}
            <span class="text-gray-600 text-lg font-normal absolute" transition:fly={{ delay: 700, duration: 500 }}>
                It's all happenning live from your browser, so it might take some time ...
            </span>
        {:else if step === 0}
            <span class="text-gray-600 text-lg font-normal absolute" transition:fly={{ duration: 500 }}>
                Drop your photo or click down below to import it.
            </span>
        {:else if isImageProcessed && step === 2}
            <span class="text-gray-600 text-lg font-normal absolute" transition:fly={{ delay: 700, duration: 500 }}>
                Edit your profile picture as you wish!
            </span>
        {/if}
    </div>
    <div class="flex justify-center items-center">
        <input 
            type="file" 
            on:change={processImage} 
            hidden 
            bind:this={fileInput}
            accept="image/png, image/jpg"
        >
        {#if !isImageProcessed && !loading}
            <div 
                class="absolute rounded-full bg-gray-950 bg-opacity-50 border-2 border-gray-200 border-dashed w-56 h-56 flex justify-center items-center p-4 text-gray-100 cursor-pointer"
                on:dragover={(event) => event.preventDefault()}
                on:dragenter={(event) => event.preventDefault()}
                on:drop={processImage}
                on:click={triggerFileInput}
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
        <div class="rounded-full w-60 h-60 overflow-hidden" style="background-color: {color}" bind:this={proPic}>
            {#if isImageProcessed}
                <img src={imageUrl} class="max-w-60 relative" style="top: {yImagePosition[0]}px; left: {xImagePosition[0]}px; transform: scale({imageZoom[0]});"/>
            {/if}
        </div>
    </div>
    {#if isImageProcessed}
        <div class="grid grid-cols-2 gap-2 mb-8 mt-8 w-96">
            <div>
                <h1 class="text-gray-700 text-xl font-semibold mb-5">Choose a colour</h1>
                <input type="color" bind:value={color}>
            </div>
            <div class="flex flex-col gap-4">
                <h1 class="text-gray-700 text-xl font-semibold mb-5">Adapt position</h1>
                <div class="flex flex-col gap-2">
                    <label>Zoom</label>
                    <Slider bind:value={imageZoom} min={0} max={3} step={0.1} />
                </div>
                <div class="flex flex-col gap-2">
                    <label>X Position</label>
                    <Slider bind:value={xImagePosition} min={-100} max={100} step={1} />
                </div>
                <div class="flex flex-col gap-2">
                    <label>Y Position</label>
                    <Slider bind:value={yImagePosition} min={-100} max={100} step={1} />
                </div>
            </div>
        </div>
        <div class="w-full flex justify-center">
            <Button style="background-color: #679c67;" on:click={downloadProPic}>Download my ProPic</Button>
        </div>
    {/if}
</main>
<footer class="absolute flex justify-center items-center gap-4 w-screen p-4 bg-green-50 h-16">
    <div class="text-gray-700 font-semibold flex items-center gap-1">
        made by <a 
            href="https://bento.me/arthurcornil" 
            class="underline flex gap-1 items-center cursor-pointer"
            target="_blank"
        >
            arthur
            <img src="./author.png" class="size-10"/>
        </a> 
    </div>
    <a 
        href="https://github.com/arthurcornil/propic" 
        target="_blank"
        class="text-gray-700 font-semibold"
    >
        <GithubSolid class="size-12"/>
    </a>
</footer>
