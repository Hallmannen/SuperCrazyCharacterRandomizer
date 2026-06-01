<script>
    import { onMount } from "svelte";
    let heads = [], hairs = [], Mouths = [], Eyes = [], Bodys = [], Pants = []
    let headIndex = 0, hairIndex = 0, MouthIndex = 0, EyeIndex = 0, BodyIndex = 0, pantIndex = 0
    $: head  = heads.length  ? "Heads/"  + heads[headIndex].name   : ""
    $: hair  = hairs.length  ? "Hairs/"  + hairs[hairIndex].name   : ""
    $: Mouth = Mouths.length ? "Mouths/" + Mouths[MouthIndex].name : ""
    $: Eye   = Eyes.length   ? "Eyes/"   + Eyes[EyeIndex].name     : ""
    $: Body  = Bodys.length  ? "Bodys/"  + Bodys[BodyIndex].name   : ""
    $: pant  = Pants.length  ? "Pants/"  + Pants[pantIndex].name   : ""
    async function load_image() {
        const load = (glob) => Promise.all(
            Object.entries(glob).map(async ([path, resolver]) => {
                const { metadata } = await resolver();
                return { name: path.split('/').pop(), ...metadata };
            })
        )
        heads  = await load(import.meta.glob("./../../static/Heads/*.png"))
        hairs  = await load(import.meta.glob("./../../static/Hairs/*.png"))
        Mouths = await load(import.meta.glob("./../../static/Mouths/*.png"))
        Eyes   = await load(import.meta.glob("./../../static/Eyes/*.png"))
        Bodys  = await load(import.meta.glob("./../../static/Bodys/*.png"))
        Pants  = await load(import.meta.glob("./../../static/Pants/*.png"))
    }
    onMount(load_image)
    function randomize() {
        headIndex  = Math.floor(Math.random() * heads.length)
        hairIndex  = Math.floor(Math.random() * hairs.length)
        EyeIndex   = Math.floor(Math.random() * Eyes.length)
        MouthIndex = Math.floor(Math.random() * Mouths.length)
        BodyIndex  = Math.floor(Math.random() * Bodys.length)
        pantIndex  = Math.floor(Math.random() * Pants.length)
    }
    async function saveSprite() {
        var canvas = document.getElementById('viewport')
        var context = canvas.getContext('2d');
        canvas.width = '192'
        canvas.height = '528'
        let bodyParts = [Body, head, hair, Eye, Mouth, pant]
        for (let i = 0; i < bodyParts.length; i++) {
            let base_image = new Image();
            base_image.src = bodyParts[i];
            base_image.addEventListener("load", () => {
                context.drawImage(base_image, 0, 0);
                if (i == bodyParts.length - 1) {
                    const dataURL = canvas.toDataURL();
                    const link = document.createElement('a');
                    link.href = dataURL;
                    link.download = 'pixel_art.png';
                    link.click();
                }
            });
        };
    }
    const step = (i, d, max) => (i + d + max) % max
</script>

<div class="absolute top-8 right-10 text-4xl font-bold text-white drop-shadow-[0    px_14px_9px_rgba(0,0,0,1)] bg-red-500 border-blue-100 border-2">
    Super crazy character randomizer
</div>

<div id="HonorWrapper">
    <h1 id="HonorTitle">---Honorable Mention---</h1>
    <img src="pixel_art (2).png" alt="Honor" id="Honor">
</div>

<button class="random" on:click={randomize}>
    <p>randomize</p>
</button>

<canvas id="viewport"></canvas>

<button class="btn bg-blue-500 border-amber-100 border-2" id="SaveTheSprite" on:click={saveSprite}>
    save sprite
</button>

<div class="face-controls">
    {#each [
        { label: "Huvud", get: () => headIndex,  set: (v) => headIndex  = v, arr: heads  },
        { label: "Ögon",  get: () => EyeIndex,   set: (v) => EyeIndex   = v, arr: Eyes   },
        { label: "Mun",   get: () => MouthIndex, set: (v) => MouthIndex = v, arr: Mouths },
        { label: "Hår",   get: () => hairIndex,  set: (v) => hairIndex  = v, arr: hairs  },
    ] as ctrl}
        <div class="face-col">
            <span>{ctrl.label}</span>
            <div class="face-row">
                <button on:click={() => ctrl.set(step(ctrl.get(), -1, ctrl.arr.length))}>&#8678;</button>
                <button on:click={() => ctrl.set(step(ctrl.get(), 1, ctrl.arr.length))}>&#8680;</button>
            </div>
        </div>
    {/each}
</div>

<main>
    <div class="side">
        <button on:click={() => BodyIndex = step(BodyIndex, -1, Bodys.length)}>&#8678;</button>
        <button on:click={() => pantIndex = step(pantIndex, -1, Pants.length)}>&#8678;</button>
    </div>
    <div class="container">
        <img src={Body} alt="body" id="body">
        <img src={pant} alt="pants">
        <img src={head} alt="head">
        <img src={hair} alt="hair">
        <img src={Eye} alt="eyes">
        <img src={Mouth} alt="mouth">
    </div>
    <div class="side">
        <button on:click={() => BodyIndex = step(BodyIndex, 1, Bodys.length)}>&#8680;</button>
        <button on:click={() => pantIndex = step(pantIndex, 1, Pants.length)}>&#8680;</button>
    </div>
</main>

<style>
canvas#viewport {
    border: 1px solid white;
    width: 900px;
    height: 15px;
    opacity: 0;
}
body {
    margin: 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
}
.random {
    display: block;
    margin: 30px auto 12px;
    background-color: azure;
    height: 70px;
    width: 200px;
    border-radius: 10px;
    border: 5px solid black;
    cursor: pointer;
}
p {
    text-align: center;
    font-size: xx-large;
    margin: 0;
}
.face-controls {
    display: flex;
    color: black;
    gap: 16px;
    justify-content: center;
    margin-bottom: 10px;
}
.face-col {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 4px;
    font-family: sans-serif;
    font-size: 0.85rem;
    font-weight: bold;
}
.face-row {
    display: flex;
    gap: 4px;
}
.face-row button,
.side button {
    width: 44px;
    height: 44px;
    font-size: 1.4rem;
    background: white;
    border: 3px solid black;
    border-radius: 6px;
    cursor: pointer;
}
.face-row button:hover,
.side button:hover { background: #e0e0e0; }
.face-row button:active,
.side button:active { background: #bbb; }
main {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 25px;
}
#SaveTheSprite {
    font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
    font-size: 24px;
    position: absolute;
    top: 300px;
    right: 250px;
}
#HonorWrapper {
    position: absolute;
    left: 40px;
    top: 120px;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 10px;
}
#HonorTitle {
    background-color: white;
    border-radius: 5px;
}
#Honor {
    background-color: lightblue;
    border: solid;
    border-radius: 20px;
    image-rendering: pixelated;
}
.container {
    position: relative;
    height: 600px;
    width: 300px;
    border: solid;
    background-color: rgb(211, 211, 211);
    overflow: hidden;
}
.container img {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    image-rendering: pixelated;
}
.side {
    display: flex;
    flex-direction: column;
    justify-content: space-around;
    height: 600px;
}
.side button {
    width: 50px;
    height: 120px;
    font-size: 2rem;
}
</style>