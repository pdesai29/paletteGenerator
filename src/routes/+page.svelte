<script>
  import { afterUpdate } from "svelte";
  import ColorCard from "./colorCard/+page.svelte";
  import { Configuration, OpenAIApi } from "openai";

  let colors = [
    { name: "Italian Sun", hex: "#FAD20F" },
    { name: "Amaranth", hex: "#E32636" },
    { name: "Mint", hex: "#48c9b0" },
    { name: "Limeade", hex: "#A8e063" },
    { name: "Venetian Red", hex: "#b82400" },
    { name: "Fiery Orange", hex: "#F57939" },
  ];
  let width = 100 / colors.length;
  let inputVal = "";
  let prevInputVal = "italy,vintage,warm,vibrant";

  let selectedColor;
  let selectedColorHex;
  const handleChange = (event) => {
    inputVal = event.target.value;
  };
  const configuration = new Configuration({
    apiKey: "Your_KEY",
  });
  const openai = new OpenAIApi(configuration);
  let prompt = `You are a color palette generating assistant that responds to text prompts for color palettes,
                    You should generate color palettes that fit the vibe,theme mood and instructions in the given prompt    
                    the palettes should be between 4 and 8 colors.

                    Example prompt:
                    Q: convert the following verbal description of a color palette into list of color description objects: The mediterranean sea
                    A: [{"name":"Italian Sun","hex":"#ffd746"},{"name":"Sicilian Orange","hex":"#f77f00"},{"name":"Tuscan Red","hex":"#7f3b08"},{"name":"Venetian Roayl","hex":"#6c3461"},{"name":"Amalfi Coast","hex":"#44a19c"},{"name":"Cinque Terre","hex":"#2a5934"}]
                    Q: convert the following verbal description of color palette into list of color description objects: Google theme colors
                    A: [{"name":"G-BLUE","hex":"#4285F4"},{"name":"G-RED","hex":"#DB4437"},{"name":"G-YELLOW","hex":"#F4B400"},{"name":"G-GREEN","hex":"#0F9D58"}]

                    Desired Format: a JSON array of objects containing name and hex field for color name and hex of color both are string

                    text: italy,vintage,warm,vibrant

                    result:
    `;
  async function getAiResponse(openai, prompt) {
    const completion = await openai.createCompletion({
      model: "text-davinci-003",
      prompt: prompt,
      max_tokens: 1024,
      n: 1,
      stop: null,
      temperature: 0.7,
    });
    return completion.data.choices[0].text;
  }
  const onClick = async () => {
    prompt = prompt.replace(prevInputVal, inputVal);
    console.log(prompt);
    try {
      colors = await getAiResponse(openai, prompt);
      console.log(colors);
      colors = JSON.parse(colors);
      console.log(typeof colors);
      width = 100 / colors.length;
      prevInputVal = inputVal;
    } catch (error) {
      if (error.response) {
        console.log(error.response.status);
        console.log(error.response.data);
      } else {
        console.log(error.message);
      }
    }
  };

  let showOverlay;
  const handleClick = (colorObject) => {
    const colorHex = colorObject.hex;
    navigator.clipboard.writeText(colorHex);
    selectedColor = colorObject.name;
    selectedColorHex = colorHex;
    showOverlay = true;
  };

  const removeOverlay = () => {
    showOverlay = false;
  };

  afterUpdate(() => {
    if (showOverlay) {
      setTimeout(removeOverlay, 1500);
    }
  });
</script>

<div class="index">
  <div
    class="overlay"
    class:hidden={!showOverlay}
    style="background-color: {selectedColorHex};"
  >
    <div class="overlay-text">
      <h1>Copied!</h1>
      <p>{selectedColorHex}</p>
      <p>{selectedColor}</p>
    </div>
  </div>
  <div class="input">
    <p style="margin-left:0px">Enter a Prompt</p>
    <input
      class="input-styled"
      type="text"
      bind:value={inputVal}
      on:input={handleChange}
    />
    <button class="btn-styled" on:click={onClick}>Get Colors</button>
  </div>

  <div class="card-container">
    {#each colors as colorObject}
      <ColorCard
        {colorObject}
        {width}
        id={colorObject.hex}
        handleCardClick={handleClick}
      />
    {/each}
  </div>
</div>

<style>
  .overlay-text {
    position: fixed;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    transform: scale(0.1);
    color: white;
    transition: all 0.4s ease-in-out;
    font-size: 20rem;
    text-shadow: 1px 2px black;
  }
  .overlay-text h1 {
    font-weight: 700;
    text-shadow: 1px 2px black;
    background: rgba(255, 255, 255, 0.2);
    width: fit-content;
    text-align: center;
    margin-bottom: 0;
    padding: 1rem;
    text-transform: uppercase;
    color: white;
    font-family: Impact, Haettenschweiler, "Arial Narrow Bold", sans-serif;
  }
  .overlay-text p {
    font-size: 15rem;
    font-weight: 500;
  }
  .overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    z-index: 9999;
    opacity: 1;
    transition: opacity 0.3s ease;
    font-family: Impact, Haettenschweiler, "Arial Narrow Bold", sans-serif;
  }

  .hidden {
    display: none;
    opacity: 0;
    pointer-events: none;
  }
  .index {
    position: relative;
    text-align: center;
    height: 100vh;
    background-color: black;
    margin: 0;
    padding: 0;
  }

  .input {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    z-index: 1;
    font-family: Impact, Haettenschweiler, "Arial Narrow Bold", sans-serif;
    color: black;
    font-size: 30px;
    border: solid black 1px;
    border-radius: 10px;
    padding: 5px;
  }
  input {
    height: 30px;
    width: 250px;
    font-family: Arial, Helvetica, sans-serif;
    margin-right: 10px;
    font-family: Impact, Haettenschweiler, "Arial Narrow", sans-serif;
    font-weight: 20;
  }

  .input-styled {
    padding: 0.6em 2em;
    border: none;
    outline: none;
    color: rgb(255, 255, 255);
    background: #111;
    position: relative;
    z-index: 0;
    border-radius: 10px;
    user-select: none;
    -webkit-user-select: none;
    touch-action: manipulation;
  }

  .btn-styled {
    padding: 0.6em 2em;
    border: none;
    outline: none;
    color: rgb(255, 255, 255);
    background: #111;
    cursor: pointer;
    position: relative;
    z-index: 0;
    border-radius: 10px;
    user-select: none;
    -webkit-user-select: none;
    touch-action: manipulation;
  }

  .btn-styled:before {
    content: "";
    background: linear-gradient(
      45deg,
      #ff0000,
      #ff7300,
      #fffb00,
      #48ff00,
      #00ffd5,
      #002bff,
      #7a00ff,
      #ff00c8,
      #ff0000
    );
    position: absolute;
    top: -2px;
    left: -2px;
    background-size: 400%;
    z-index: -1;
    filter: blur(5px);
    -webkit-filter: blur(5px);
    width: calc(100% + 4px);
    height: calc(100% + 4px);
    animation: glowing-btn-styled 20s linear infinite;
    transition: opacity 0.3s ease-in-out;
    border-radius: 10px;
  }

  @keyframes glowing-btn-styled {
    0% {
      background-position: 0 0;
    }
    50% {
      background-position: 400% 0;
    }
    100% {
      background-position: 0 0;
    }
  }

  .btn-styled:after {
    z-index: -1;
    content: "";
    position: absolute;
    width: 100%;
    height: 100%;
    background: #222;
    left: 0;
    top: 0;
    border-radius: 10px;
  }

  .card-container {
    display: flex;
    justify-content: flex-start;
    z-index: 0;
    margin: auto 0;
  }

  button {
    height: 40px;
    width: 150px;
  }
</style>
