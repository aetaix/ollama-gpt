<script>
  import ollama from "ollama";
  import { browser } from "$app/environment";
  import { goto } from "$app/navigation";
  import { currentModel, models } from "$lib/stores/models";

  export let name = "";
  export let image = "";
  export let icon = "";
  export let parameters = "";
  export let description = "";
  export let tags = [];
  export let size = "";
  export let installed = false;

  let sizeColor =
    size < 2 ? "bg-green-100" : size < 5 ? "bg-orange-100" : "bg-red-100";

  /**
   * Model I/O
   */

  let loading = false;

  $: {
    if (browser) {
      // check local storage if there is an installation queue running
      const queue = JSON.parse(localStorage.getItem("queue") || "[]");
      // if the model is in the queue and is not already installed, set loading to true esle remove it from the queue
      if (queue.includes(image) && !installed) {
        loading = true;
      } else {
        localStorage.setItem(
          "queue",
          JSON.stringify(queue.filter((item) => item !== image))
        );
      }
   
    }
  }

  function setCurrentModel() {
    currentModel.set({ name, image, icon, description, tags, size, installed });
    goto("/");
  }

  async function installModel() {
    loading = true;

    // Add the model to the installation queue
    if (browser) {
      const queue = JSON.parse(localStorage.getItem("queue") || "[]");
      localStorage.setItem("queue", JSON.stringify([...queue, image]));
    }
    // Use fetch on api/pull-model to install the model
    const response = await fetch("/api/pull-model", {
      method: "POST",
      body: JSON.stringify({ image }),
    });
  }

  function deleteModel() {
    ollama.delete({ model: image }).then(() => {
      models.update((models) =>
        models.filter((model) => model.image !== image)
      );
    });
  }
</script>

<div
  class="bg-white dark:bg-black-700 border border-black-200 hover:shadow-lg transition-shadow rounded-xl p-4"
>
  <header class="flex items-center justify-between mb-4">
    <div class="flex items-center gap-2">
      <img src="/icons/models/{icon}" alt={name} class="w-4 h-4" />

      <h2 class="text-sm">
        <span class="font-semibold">{name}</span><span
          class="font-mono ml-1 text-black-500">{parameters}</span
        >
      </h2>
    </div>

    {#if installed}
      <div class="flex items-center justify-end gap-2">
        <button
          on:click={setCurrentModel}
          class="border bg-green-500 border-green-300 text-white shadow px-2 py-1 rounded-md text-sm"
        >
          Chat
        </button>
        <button
          on:click={deleteModel}
          class="border border-black-200 bg-white px-2 py-1 rounded-md text-sm"
        >
          Delete
        </button>
      </div>
    {:else if loading}
      <div class="flex items-center gap-2">
        <svg width="20" stroke="#000" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><g class="spinner_V8m1"><circle cx="12" cy="12" r="9.5" fill="none" stroke-width="3" class=" stroke-green-500"></circle></g></svg>
        <span class="text-xs ">Installing...</span>
      </div>
    {:else}
      <button
        on:click={installModel}
        class="border border-black-200 bg-white px-2 py-1 rounded-md text-sm"
      >
        Install
      </button>
    {/if}
  </header>
  <p class="text-sm text-black-500">{description}</p>
  <footer class="flex items-center justify-between mt-4">
    <div class="flex items-center gap-1">
      {#each tags as tag}
        <span
          class="text-xs text-black-500 dark:text-black-300 border border-black-200 px-2 py-1 rounded-full"
          >{tag}</span
        >
      {/each}
    </div>
    <span
      class="text-xs text-black-500 {sizeColor} dark:text-black-300 px-2 py-1 rounded-full"
      >{size}Go</span
    >
  </footer>
</div>

<style>.spinner_V8m1{transform-origin:center;animation:spinner_zKoa 2s linear infinite}.spinner_V8m1 circle{stroke-linecap:round;animation:spinner_YpZS 1.5s ease-in-out infinite}@keyframes spinner_zKoa{100%{transform:rotate(360deg)}}@keyframes spinner_YpZS{0%{stroke-dasharray:0 150;stroke-dashoffset:0}47.5%{stroke-dasharray:42 150;stroke-dashoffset:-16}95%,100%{stroke-dasharray:42 150;stroke-dashoffset:-59}}</style>