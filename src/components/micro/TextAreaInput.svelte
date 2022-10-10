<script lang="ts">
  export let handleSelection;
  export let value;
</script>

<textarea
  readonly
  class="textarea h-24 max-h-96 w-full bg-gray-100 text-base selection:bg-red-500 selection:text-white text-slate-600 rounded mt-1 mb-1 pl-2 border-slate-300 text-left"
  rows="1"
  on:select={(e) => {
    handleSelection(
      e.currentTarget.selectionStart,
      e.currentTarget.selectionEnd
    );
  }}
  {value}
/>

<style>
  .textarea {
    resize: none;
  }
</style>
