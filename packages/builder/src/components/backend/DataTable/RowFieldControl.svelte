<script>
  import {
    Input,
    Select,
    DatePicker,
    Toggle,
    Multiselect,
    Label,
    RichTextField,
    TextArea,
  } from "@budibase/bbui"
  import Dropzone from "components/common/Dropzone.svelte"
  import { capitalise } from "helpers"
  import LinkedRowSelector from "components/common/LinkedRowSelector.svelte"
  import Editor from "../../integration/QueryEditor.svelte"

  export let defaultValue
  export let meta
  export let value = defaultValue || (meta.type === "boolean" ? false : "")
  export let readonly

  const resolveTimeStamp = timestamp => {
    if (!timestamp) {
      return null
    }
    let maskedDate = new Date(`0-${timestamp}`)
    if (maskedDate instanceof Date && !isNaN(maskedDate.getTime())) {
      return maskedDate
    } else {
      return null
    }
  }

  $: stringVal =
    typeof value === "object" ? JSON.stringify(value, null, 2) : value
  $: type = meta?.type
  $: label = meta.name ? capitalise(meta.name) : ""

  const timeStamp = resolveTimeStamp(value)
  const isTimeStamp = !!timeStamp
</script>

{#if type === "options" && meta.constraints.inclusion.length !== 0}
  <Select
    {label}
    data-cy="{meta.name}-select"
    bind:value
    options={meta.constraints.inclusion}
    sort
  />
{:else if type === "datetime"}
  <DatePicker {label} timeOnly={isTimeStamp} bind:value />
{:else if type === "attachment"}
  <Dropzone {label} bind:value />
{:else if type === "boolean"}
  <Toggle text={label} bind:value data-cy="{meta.name}-input" />
{:else if type === "array" && meta.constraints.inclusion.length !== 0}
  <Multiselect bind:value {label} options={meta.constraints.inclusion} />
{:else if type === "link"}
  <LinkedRowSelector bind:linkedRows={value} schema={meta} />
{:else if type === "longform"}
  {#if meta.useRichText}
    <RichTextField {label} height="150px" bind:value />
  {:else}
    <TextArea {label} height="150px" bind:value />
  {/if}
{:else if type === "json"}
  <Label>{label}</Label>
  <Editor
    editorHeight="250"
    mode="json"
    on:change={({ detail }) => (value = detail.value)}
    value={stringVal}
  />
{:else}
  <Input
    {label}
    data-cy="{meta.name}-input"
    {type}
    bind:value
    disabled={readonly}
  />
{/if}
