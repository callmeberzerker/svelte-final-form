<script>
  import { getContext, onDestroy } from "svelte";
  import { fieldSubscriptionItems } from "final-form";
  import { FORM } from "svelte-final-form/src/Form.svelte";

  export let name,
    subscription = getFieldSubscriptionItems(),
    validate = undefined,
    parse = undefined;

  let meta = {};
  let input = {};
  let unsubscribe;

  const form = getContext(FORM);

  if (process.env.NODE_ENV !== "production" && !form) {
    throw new Error(
      "Could not find svelte-final-form context value. Please ensure that your Field is inside the Form component."
    );
  }

  $: {
    unsubscribe && unsubscribe();

    unsubscribe = form.registerField(
      name,
      (fieldState) => {
        const { blur, change, focus, value, ...fieldMeta } = fieldState;

        meta = fieldMeta;

        input = {
          name,
          onBlur: blur,
          onChange: (val) => change(parse ? parse(val) : val),
          onFocus: focus,
          value,
        };
      },
      subscription,
      {
        getValidator: () => validate,
      }
    );
  }

  onDestroy(() => {
    unsubscribe && unsubscribe();
  });

  function getFieldSubscriptionItems() {
    return fieldSubscriptionItems.reduce((result, key) => {
      result[key] = true;
      return result;
    }, {});
  }
</script>

<slot {meta} {input} />
