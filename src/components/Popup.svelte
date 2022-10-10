<script lang="ts">
  import { orgHero } from "src/stores/context";
  import { onMount } from "svelte";
  import TextAreaInput from "./micro/TextAreaInput.svelte";

  const doesNotContainSpace = (text) => {
    if (!text.includes(" ")) return true;
    return false;
  };

  const hasValidLength = (text) => {
    if (text.length >= 1) return true;
    return false;
  };

  const isValidShortLink = (text) => {
    return hasValidLength(text) && doesNotContainSpace(text);
  };

  chrome.runtime.onMessage.addListener((msg) => {
    if (msg.type === "create_link_response") {
      isCreatingShortLink = false;
      if (msg.status === "success") {
        successMessage = "Success! You can now slash/go to your new shortlink";
      } else {
        errorMessage = msg.message;
      }
      selectedType = `static`;
    }
  });

  let isCreatingShortLink: boolean = false;
  let currentPageUrl = "";

  let successMessage: string = null,
    errorMessage: string = null;
  let shortLink: string = "",
    isPrivate: boolean = false;
  let selectedType: "dynamic" | "static" = `static`;
  let dynamicUrl: string = "";

  $: isSubmitDisabled = !isShortlinkValid || isCreatingShortLink;
  $: isShortlinkValid = isValidShortLink(shortLink);

  const privatePath = `my/`;
  $: displayPath = isPrivate ? `${$orgHero}/${privatePath}` : `${$orgHero}/`;

  $: displayPathSuffix = selectedType === "dynamic" ? "/<var>" : "";

  let shortLinkInput;

  const createLink = async (url: string) => {
    isCreatingShortLink = true;
    errorMessage = "";
    successMessage = "";

    chrome.runtime.sendMessage({
      command: "create_link",
      data: { url, shortLink, selectedType, isPrivate },
    });
  };

  $: submit = async () => {
    chrome.tabs.query({ active: true }, (tabs) => {
      console.info(`creating shortlink for ${tabs[0].url}`);
      createLink(selectedType === `dynamic` ? dynamicUrl : tabs[0].url);
    });
  };

  onMount(() => {
    chrome.tabs.query({ active: true }, (tabs) => {
      currentPageUrl = tabs[0].url;
    });
    shortLinkInput.focus();
  });
</script>

<section class="antialiased bg-gray-100 text-gray-600 h-screen px-4">
  <div class="flex flex-col h-full w-96 ">
    <button
      class="absolute underline right-0 top-0 p-2 text-sm "
      on:click={() => {
        chrome.runtime.openOptionsPage();
      }}>All links</button
    >
    <div class=" main py-8 px-6 pt-12 bg-slate-100">
      <div class="p-2">
        <form on:submit|preventDefault={submit}>
          <div>
            <div class="flex">
              <label for="shortlink" class="mb-1 block text-sm"
                >Create a shortlink for current page:</label
              >
              <!-- <div
                class="border-r-0 select-none font-bold text-slate-500 mt-1 mb-1 p-2 text-lg border border-slate-300 rounded-l-md text-left"
              >
                {displayPath}
              </div> -->
              <input
                id="shortlink"
                bind:this={shortLinkInput}
                bind:value={shortLink}
                type="input"
                class="text-lg text-center w-full px-2 text-slate-500 border border-slate-300 "
                placeholder="shortlink"
                disabled={isCreatingShortLink}
              />
              <!-- <div
                class="border-x-0 select-none font-bold text-slate-500 mt-1 mb-1 p-2 text-lg border border-slate-300 rounded-r-md text-left"
              >
                {displayPathSuffix}
              </div> -->
            </div>
            <label
              for="shortlink"
              class="block text-xs mt-1 text-center text-slate-400 "
              >Tip: Best shortlinks are short and memorable</label
            >
          </div>

          {#if isShortlinkValid}
            <details class="mt-6 tex-sm">
              <summary class="text-xs">Advanced Settings:</summary>
              <div class=" border space-y-2 rounded-lg px-2 py-1 text-sm  ">
                <div>
                  <div class="flex cursor-pointer">
                    <label class="w-full" for="is-private"
                      >make it private?</label
                    >
                    <input
                      id="is-private"
                      bind:checked={isPrivate}
                      disabled={isSubmitDisabled}
                      type="checkbox"
                    />
                  </div>
                </div>
                <div>
                  <div class="flex cursor-pointer">
                    <label class="w-full" for="is-dynamic"
                      >make it dynamic?</label
                    >
                    <input
                      id="is-dynamic"
                      type="checkbox"
                      checked={selectedType === "dynamic"}
                      on:click={(_) => {
                        selectedType === "static"
                          ? (selectedType = "dynamic")
                          : (selectedType = "static");
                      }}
                      disabled={isSubmitDisabled}
                    />
                  </div>
                  {#if selectedType == `dynamic`}
                    <div class="mt-2">
                      <div class="text-xs text-red-500">
                        Select part of url that you want to make Dynamic
                      </div>
                      <!-- <input
                bind:value={dynamicUrl}
                type="input"
                class="w-full text-lg text-slate-600 rounded mt-1 mb-1 pl-2 border border-slate-300 text-left"
                placeholder="shortlink"
              /> -->
                      <TextAreaInput
                        handleSelection={(start, end) => {
                          console.log(start, end);
                          dynamicUrl =
                            currentPageUrl.slice(0, start) +
                            "<var>" +
                            currentPageUrl.slice(end);
                          console.log(dynamicUrl);
                        }}
                        value={currentPageUrl}
                      />
                    </div>
                  {/if}
                </div>
              </div>
            </details>

            <div class="mt-6 text-xs tex-center ">
              <div class="h-10 w-10 absolute right-1/2">
                <!-- <Arrow /> -->
              </div>
              <!-- <div class="text-xs">Preview:</div> -->
              <div class="select-none preview border p-1 text-center">
                <div class="flex items-center">
                  <div class=" w-5/12">
                    <!-- <div>If you search for:</div> -->
                    <code
                      class="break-words bg-gray-200  my-1 text-slate-600 select-all text-sm"
                      >{`go/${isPrivate ? "my/" : ""}${shortLink}${
                        selectedType === "dynamic" ? "/<var>" : ""
                      }`}</code
                    >
                  </div>
                  <div class="w-7/12">
                    <!-- <div>then will be navigated to:</div> -->
                    <div class="my-1 break-words">
                      {`${
                        selectedType === "dynamic" ? dynamicUrl : currentPageUrl
                      }`}
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <div class="w-100 text-center mt-6">
              <button
                disabled={isSubmitDisabled ||
                  (selectedType === "dynamic" && dynamicUrl === "")}
                class="disabled:opacity-50 mt-2 w-60 mb-2 bg-red-500 hover:bg-red-600 text-white font-bold pt-2 pb-2 pl-6 pr-4 rounded"
                type="submit"
                >{isCreatingShortLink ? "Creating..." : "Create"}</button
              >
            </div>
          {/if}
        </form>
        {#if successMessage}<div
            class="p-2 text-center text-green-500 font-bold"
          >
            {successMessage}
          </div>{/if}
        {#if errorMessage}<div class="p-2 text-center text-red-500 font-bold">
            {errorMessage}
          </div>{/if}
      </div>
    </div>
  </div>
</section>
