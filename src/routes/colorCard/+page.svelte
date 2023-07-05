<script>

   export let colorObject;
   export let width;
   export let id;
   export let handleCardClick;
   export let handleKeyDown=(event)=> {
    if (event.key === "Enter" || event.key === "Space") {
      handleCardClick();
    }
  }
</script>
<div class="card" role="button" tabindex="0" style="background-color: {colorObject.hex};width:{width}vw" id={id} on:click={()=>handleCardClick(colorObject)} on:keydown={handleKeyDown}>
    <p>{colorObject.name}</p>
    <p>{colorObject.hex}</p>
</div>

<style>
.card {
  height: 100vh;
  width: 50px;
  float: left;
  margin: auto 0;
  font-family: Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif;
  font-size: x-large;
  color:whitesmoke;
  box-shadow: 4px 4px 4px 4px rgba(0, 0, 0, 0.2);

}


</style>