<script>
  import Nav from "./Nav.svelte";
  import Bioformats2rawLayout from "./Bioformats2rawLayout/index.svelte";
  import JsonValidator from "./JsonValidator/index.svelte";
  import Title from "./Title.svelte"
  import Modal from "svelte-simple-modal";

  import { getJson } from "./utils";

  const searchParams = new URLSearchParams(window.location.search);
  let source = searchParams.get("source");
  if (source && !source.endsWith("/")) {
    source = source + "/";
  }

  let location = window.location.href;

  let promise;

  if (source) {
    // load JSON to be validated...
    console.log("Loading JSON... " + source + ".zattrs");
    promise = getJson(source + ".zattrs");
  }
</script>

<Modal>
  <main>
    <nav>
      <Nav />
    </nav>
    <section>
      {#if source}
        {#await promise}
          <div>loading...</div>
        {:then data}
          <Title {source} zattrs={data} />
          <div>
            {#if data["bioformats2raw.layout"] === 3 && !data.plate}
              <Bioformats2rawLayout rootAttrs={data} {source} />
            {:else}
              <JsonValidator rootAttrs={data} {source} />
            {/if}
          </div>
        {:catch error}
          <p style="color: red">{error.message}</p>
        {/await}
      {:else}
        <article>
          To validate an OME-ZARR file, use e.g.
          <a
            href="{location}?source=https://uk1s3.embassy.ebi.ac.uk/idr/zarr/v0.3/9836842.zarr"
          >
            ?source=https://uk1s3.embassy.ebi.ac.uk/idr/zarr/v0.3/9836842.zarr
          </a>
        </article>
      {/if}
    </section>
  </main>
</Modal>

<style>
  main {
    display: flex;
    flex-direction: column;
    height: 100%;
  }
  nav {
    display: block;
    margin: 0;
    padding: 0;
    flex: 0 0 48px;
    background-color: #343a40;
  }

  section {
    overflow: auto;
    width: 100%;
    flex: 1;
    padding: 15px 0;
    background: #ff512f; /* fallback for old browsers */
    background: -webkit-linear-gradient(
      to right,
      #f09819,
      #ff512f
    ); /* Chrome 10-25, Safari 5.1-6 */
    background: linear-gradient(
      to right,
      #f09819,
      #ff512f
    ); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
    /* https://uigradients.com/ */
    /* OME green->blue rgb(61,132,107), rgb(68,139,200) */
  }

  article {
    width: 90%;
    margin: auto;
  }

  section > div {
    margin: auto;
    width: clamp(300px, 95%, 1200px);
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    justify-content: center;
    gap: 20px;
  }

  @media (min-width: 1000px) {
    section > div {
      flex-direction: row;
    }

    article {
      width: 60%;
      margin: auto;
    }
    a {
      white-space: nowrap;
    }
  }
</style>
