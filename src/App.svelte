<script>
  import OpenAI from "openai";
  import { get } from "svelte/store";

  let inputText = "";
  let isLoading = false;
  let responseText;
  let image_url;
  const config = {
    apiKey: import.meta.env.VITE_OPENAI_API_KEY,
    dangerouslyAllowBrowser: true,
  };
  const openai = new OpenAI(config);

  async function runPrompt() {
    image_url = null;
    try {
      isLoading = true;

      const prompt = `
        Find two suitable animals to phonetically recreate a human first name. 
        For example, the name “Fabian” could be derived from the animal species names “pheasant” and “baboon”, “Pius” could have been formed from “puma” and “ibis”. 
        Find two other animal species that, taken together and mixed, would result in the name ${inputText}. `;
      const response = await openai.chat.completions.create({
        model: "gpt-4o-mini",
        messages: [
          { role: "user", content: prompt },
          {
            role: "system",
            content:
              "You analyze first names and return the names of animals as their parents.",
          },
        ],
        response_format: {
          type: "json_schema",
          json_schema: {
            name: "animals_schema",
            schema: {
              type: "object",
              properties: {
                mother: {
                  description: "The name of an animal species",
                  type: "string",
                },
                father: {
                  description: "The name of an animal species",
                  type: "string",
                },
              },
              additionalProperties: false,
            },
          },
        },
      });

      console.log("API-Response:", response);
      responseText = response.choices[0].message.content;
      getImage(
        JSON.parse(responseText).mother,
        JSON.parse(responseText).father,
      );
    } catch (error) {
      // console.error("Error while retrieving the dinosaur name:", error);
    } finally {
      isLoading = false;
    }
  }

  async function getImage(mother, father) {
    const response = await openai.images.generate({
      model: "dall-e-3",
      prompt: "an animal that is a mix of a " + mother + " and a " + father,
      n: 1,
      size: "1024x1024",
    });
    image_url = response.data[0].url;
  }
</script>

<main>
  <h1>Ancestor Animalysis</h1>
  <div class="input-controls">
    <input
      type="text"
      bind:value={inputText}
      placeholder="Enter a name or thing..."
    />
    <button on:click={runPrompt} disabled={isLoading}>Research!</button>
  </div>
  {#if isLoading}
    <p>Unveiling your primal lineage...</p>
  {:else if responseText}
    <p>Your animal ancestry revealed:</p>
    <p class="result">Mother: {JSON.parse(responseText).mother}</p>
    <p class="result">Father: {JSON.parse(responseText).father}</p>
    {#if image_url}
      <img src={image_url} alt="Ancestor Animal" />
    {/if}
  {/if}
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    align-items: center;
    height: 100vh;
    font-family: "Lucida Sans", "Lucida Sans Regular", "Lucida Grande",
      "Lucida Sans Unicode", Geneva, Verdana, sans-serif;
    background-color: #222;
    color: #fff;
  }

  .input-controls {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    gap: 20px;
    margin-bottom: 2em;
  }

  input {
    width: 200px;
    border-radius: 5px;
    border: 1px solid #ccc;
    padding: 10px;
  }

  button {
    border-radius: 5px;
    border: 1px solid #ccc;
    padding: 10px;
  }
  p {
    font-size: 0.7rem;
  }
  .result {
    margin-top: 0;
    font-size: 1.5rem;
    font-weight: bold;
    max-width: 500px;
  }
  img {
    max-height: 500px;
  }
</style>
