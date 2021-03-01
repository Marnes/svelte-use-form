# svelte-use-form

A svelte form library that is easy to use and has 0 boilerplate. It helps you control and validate forms and their inputs and check on the state of them.

Features:

- Supports: Inputs, TextAreas, Selects, Radio Buttons, Checkboxes...
- Uses single object to represent the state, instead of splitting it up (errors, values, controls...)
- Requires no special binding on the Form Input Elements, only the name attribute
- OTB and custom validators
- No requirement to use custom components
- Dynamic Inputs => Show / Hide Inputs at runtime.

## Installation

```bash
# npm
npm i -D svelte-use-form

# yarn
yarn add -D svelte-use-form
```

## Usage

It's pretty self-explanatory, just check out the examples below 😉<br>
Just make sure to prefix the form with `$` , when accessing its state.

#### REPLs:

- [Registration](https://svelte.dev/repl/a6665267d7d0435ebc7921a250552a25?version=3.34.0)
- [Testing the limits](https://svelte.dev/repl/d4fc021f688d4ad0b3ceb9a1c44c9be9?version=3.34.0)

### Minimal Example

```html
<script>
  import { useForm, minLength } from "svelte-use-form";

  const form = useForm({ title: { validators: [minLength(5)] } });
</script>

<form use:form>
  <input name="title" />
  <button disabled="{!$form.valid}">Submit</button> <br />

  <Hint name="title" on="minLength" let:value>
    The title requires at least {value} characters.
  </Hint>
</form>
```

or you can also print the error message like this:

```html
...
  {#if $form.title.touched && $form.title.errors.minLength}
    The title requires at least {$form.title.errors.minLength} characters.
  {/if}
</form>

```

### Login Example (Styling omitted)

```html
<script>
  import { useForm, email, required } from "svelte-use-form";
  const form = useForm({
    email: { validators: [email, required] },
    password: { validators: [required] },
  });
</script>

<form use:form>
  <h1>Login</h1>
  <input type="email" name="email" />

  <HintGroup name="email">
    <Hint on="required">This is a mandatory field</Hint>
    <Hint on="email" hideWhenRequired>Email is not valid</Hint>
  </HintGroup>

  <input type="password" name="password" />
  <Hint name="password" on="required">This is a mandatory field</Hint>

  <button disabled="{!$form.valid}">Login</button>
</form>
```

## API

### const newForm = useForm(properties)

useForm() returns a svelte `store` (Observable) that is also an `action`. (That's what I call [svelte](https://www.dictionary.com/browse/svelte) 😆)<br>

#### properties

- { [name_of_input: string]: {initial?: string, validators: Validator[] }
  - initial = the initial value for the input
  - validators
    e.g. useForm({ firstname: { initial: "John", validators: [required] } })

#### newForm

Returns an `action` that can be used on a form. It binds the form state to the form element.

#### $newForm

Subscribe to the form with `$` prefix to access the state of the form. It returns a `Form` instance.

### Form

- valid: boolean
- touched: boolean
- values: { [formControlName]: value } > Returns an object with the keys and the values being the name of the FormControl and its value.
- `[formcontrol]: FormControl`

Every input in the form will be accessible through the form directly. e.g. `<input name="email" />` === $form.email

### FormControl

- value: string
- valid: boolean
- validate: function
- touched: boolean
- errors: {[errorName: string]: any}

### Hint

Properties:

- name="name_of_input"
- on="error" > the error which should trigger it
- untouched > hint will get displayed even if the field hasn't been touched yet.
- hideWhen="different_error" > hides the hint if the different error is throwing
- hideWhenRequired > shortcut for hideWhen="required"
- let:value > returns the value of the error

### HintGroup

Properties:

- name="name_of_input"

You can omit the Hint "name" prop when wrapping it with a HintGroup.

### Validators

- required
- minLength(n)
- maxLength(n)
- number
- email

#### Custom Validator

A validator needs to be a function that returns null if valid else an object with the key being the name of the error. The value of the object will be accessible through the error. e.g. $form.title.errors.name_of_error -> 'info'.

```typescript
function passwordMatch(value: string, form: Form): null | ValidationErrors {
	return value !== form.password.value ? null : { passwordMatch: "Passwords are not matching" }
}

... passwordConfirmation: { validators: [passwordMatch] } }

... $form.title.errors.passwordMatch
```

## Note

### Chrome Autofill

When Chrome autofills the form on page load, it will register all inputs as valid. After clicking anywhere on the site, pressing a key or pressing the submit button it will validate all fields and set the correct state of the form. Note that when the user triggers a submit event, it will not fire if the fields are invalid. This solution was needed due to Chromes way of autofilling forms without really filling the inputs with values, until the page gets a click or key event.
