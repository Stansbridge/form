# rc-form

React High Order Form Component.

[![NPM version][npm-image]][npm-url]
[![build status][travis-image]][travis-url]
[![Test coverage][coveralls-image]][coveralls-url]
[![gemnasium deps][gemnasium-image]][gemnasium-url]
[![node version][node-image]][node-url]
[![npm download][download-image]][download-url]

[npm-image]: http://img.shields.io/npm/v/rc-form.svg?style=flat-square
[npm-url]: http://npmjs.org/package/rc-form
[travis-image]: https://img.shields.io/travis/react-component/form.svg?style=flat-square
[travis-url]: https://travis-ci.org/react-component/form
[coveralls-image]: https://img.shields.io/coveralls/react-component/form.svg?style=flat-square
[coveralls-url]: https://coveralls.io/r/react-component/form?branch=master
[gemnasium-image]: http://img.shields.io/gemnasium/react-component/form.svg?style=flat-square
[gemnasium-url]: https://gemnasium.com/react-component/form
[node-image]: https://img.shields.io/badge/node.js-%3E=_0.10-green.svg?style=flat-square
[node-url]: http://nodejs.org/download/
[download-image]: https://img.shields.io/npm/dm/rc-form.svg?style=flat-square
[download-url]: https://npmjs.org/package/rc-form

## Development

```
npm install
npm start
```

## Example

http://localhost:8000/examples/

online example: http://react-component.github.io/form/examples/

## Feature

* support reactjs and even react-native

## Install

[![rc-form](https://nodei.co/npm/rc-form.png)](https://npmjs.org/package/rc-form)

## Usage

```js
import { createForm } from 'rc-form';

class Form extends React.Component {
  submit = () => {
    this.props.form.validateFields((error, value) => {
      console.log(error, value);
    });
  }

  render() {
    let errors;
    const { getFieldProps, getFieldError } = this.props.form;
    return (<div>
      <input {...getFieldProps('normal')}/>
      <input {...getFieldProps('required', {
        onChange(){}, // have to write original onChange here if you need
        rules: [{required: true}],
      })}/>
      {(errors = getFieldError('required')) ? errors.join(',') : null}
      <button onClick={this.submit}>submit</button>
    </div>)
  }
}

export createForm()(Form);
```

or a quicker version:

```js
import { createForm } from 'rc-form';

class Form extends React.Component {
  componentWillMount() {
    this.requiredDecorator = this.props.form.getFieldDecorator('required', {
        rules: [{required: true}],
    });
  },

  submit = () => {
    this.props.form.validateFields((error, value) => {
      console.log(error, value);
    });
  }

  render() {
    let errors;
    const { getFieldError } = this.props.form;
    return (<div>
      {this.requiredDecorator(
      <input onChange={
      // can still write your own onChange }
      />)}
      {(errors = getFieldError('required')) ? errors.join(',') : null}
      <button onClick={this.submit}>submit</button>
    </div>)
  }
}

export createForm()(Form);
```

## `createForm(formOption: Object): Function`

- **Parameters**
	- `formOption` (Optional)

		Option Key | Signature | Description
		---|---|---
		`validateMessages `| `Object` | Preset messages of [async-validator](https://github.com/yiminghe/async-validator)
		`mapProps` | `Function(props): Object` | Get new props transfered to WrappedComponent. Defaults to props=>props.
		`onFieldsChange` | `Function(props, changedFields)` | Called when field changed, you can dispatch fields to redux store.
		`onValuesChange` | `Function(props, changedValues)` | Called when value changed.
		`mapPropsToFields` | `Function(props)` | convert value from props to fields. used for read fields from redux store.
		`withRef` | `Boolean` | Maintain an ref for wrapped component instance, use `refs.wrappedComponent` to access.


__CreateForm() will return another function which will pass an object as prop `form` with the following members to `WrappedComponent`:__

### `getFieldProps(name: String, option: Object): Object`
Will create props which can be set on any Component which supports both the `value` and `onChange` interface. Once set, this will create a binding between the form logic and the recipient input.

- **Parameters**
	- `name: String` (Required): The unique name for the input
	- `option: Object` (Optional):
	
		Option Key | Signature | Description
		---|---|---
		`exclusive` | `Boolean` | Whether to set the value exclusively. Commonly used with mutually exclusive inputs such as `<input type="radio" />`.
		`valuePropName` | `String` | Prop name of component's value field, eg: checkbox should be set to `checked`.
		`getValueFromEvent` | `Function` | Specify how to get value from the arguments to `onChange`. Commonly an event object.
		`getValueProps` | `Function` | Specify how to get the component props according to the field value.
		`initialValue` | `mixed` | The initial value for the `value` prop of the current component.
		`normalize` | `Function(value, prevValue, allValues)` | Return normalized value.
		`trigger` | `String` | Defaults to `onChange`. This is the name of the prop which is listened to collect form data.
		`validateTrigger` | `String | String[]` | Event which is listened to validate. Defaults to `onChange`. Set to `false` to only validate when calling prop `validateFields.`
		`validate` | `Object[]` | An array containing validation rule objects. (See below)
		`validateFirst` | `Boolean` | Defaults to `false`. Whether to stop validate on the first error for this field.
		`fieldNameProp` | | Where to store the `name` argument of `getFieldProps`
		`fieldMetaProp` | | Where to store the meta data of `getFieldProps`

- **Validation Rule Objects**
	
	Option Key | Signature | Description
	---|---|---
	`trigger` | `String | String[]` | Event which is listened to validate on a per rule basis, see `validateTrigger` above
	`rules` | `Object[]` | Validator rules. see: [async-validator](https://github.com/yiminghe/async-validator)
	
- **Examples**
```jsx
<form>
  <input {...getFieldProps('name', { ...options })} />
</form>
```

`getValueFromEvent` Defaults to:

```js
function (e) {
  if (!e || !e.target) {
    return e;
  }
  const { target } = e;
  return target.type === 'checkbox' ? target.checked : target.value;
}
```

`getValueProps` Defaults to:
```js
function (value) {
  return { value };
}
```

`validateTrigger` and `rules`:
```js
{
  validateTrigger: 'onBlur',
  rules: [{required: true}],
}
// is the shorthand of
{
  validate: [{
    trigger: 'onBlur',
    rules: [required: true],
  }],
}
```


### `getFieldDecorator(name:String, option: Object): (React.Node): React.Node`
Similar to `getFieldProps`, but add some helper warnings and you can write the `trigger` (`onChange` / `onFoo`) prop directly inside React.Node props:

- **Parameters**
	See `getFieldProps`
	
- **Example**

```jsx
<form>
  {getFieldDecorator('name', otherOptions)(<input />)}
</form>
```

### `getFieldValue(fieldName: String)`
Get field value by fieldName.

- **Parameters**
	- `fieldName: String`: The field name to return the value of

### `getFieldsValue(fieldNames: String[])`
Get fields value by fieldNames. See `getFieldValue`

- **Parameters**
	- `fieldNames: String[]`: The field names to return the values of

### `getFieldInstance(fieldName: String)`

- **Parameters**
	- `fieldName: String`: The field name to return the instance of

### `setFieldsValue(obj: Object)`
Set fields value by a key:value object

- **Parameters**
	- `obj: Object`:  key:value where the object keys match the fieldName

### `setFieldsInitialValue(obj: Object)`
Set fields initivalValue by a key:value object,

- **Parameters**
	- `obj: Object`:  key:value where the object keys match the fieldName. Used for reset and the initial display value.

### `setFields(obj: Object)`
Set fields by key:value object

- **Parameters**
	- `obj: Object`:  key:value where the object keys match the fieldName. Each field can contain `errors` and `value` object member.

### `validateFields([fieldNames: String[]], [options: Object], callback: Function(errors, values))`
Validate and get fields value by fieldNames.



And add `force` and `scroll`. `scroll` is the same as [dom-scroll-into-view's function parameter `config`](https://github.com/yiminghe/dom-scroll-into-view#function-parameter).

- **Parameters**
	- `fieldNames: String[]`: An array of field names
	- `options: Object`: Available values are the same as the `validate` method of [async-validator](https://github.com/yiminghe/async-validator). `force` Defaults to false and determines whether to validate fields which have already been validated via `validateTrigger`.
	- `callback: Function(errors, values)`: Callback when async validatioin is complete


### `getFieldsError(names): Object{ [name]: String[] }`

Get inputs' validate errors.

### `getFieldError(name): String[]`

Get input's validate errors.

### `isFieldValidating(name: String): Bool`

Whether this input is validating.

### `isFieldsValidating(names: String[]): Bool`

Whether one of the inputs is validating.

### `isFieldTouched(name: String): Bool`

Whether this input's value had been change.

### `isFieldsTouched(names: String[]): Bool`

Whether one of the inputs' values had been change.

### `isSubmitting(): Bool`

Whether the form is submitting.

### `submit(callback: Function)`

Cause isSubmitting to return true, after callback called, isSubmitting return false.

### `resetFields([names: String[]])`

Reset specified inputs. Defaults to all.


## `rc-form/lib/createDOMForm(formOption): Function`

createForm enhancement, support props.form.validateFieldsAndScroll

### `props.form.validateFieldsAndScroll([fieldNames: String[]], [options: Object], callback: Function(errors, values))`

props.form.validateFields enhancement, support scroll to the first invalid form field

#### `options.container: HTMLElement`

Defaults to first scrollable container of form field(until document).


## Notes

- Do not use stateless function component inside Form component: https://github.com/facebook/react/pull/6534

- you can not set same prop name as the value of validateTrigger/trigger for getFieldProps

```js
<input {...getFieldProps('change',{
  onChange: this.iWantToKnow // you must set onChange here or use getFieldDecorator to write inside <input>
})}>
```

- you can not use ref prop for getFieldProps

```js
<input {...getFieldProps('ref')} />

this.props.form.getFieldInstance('ref') // use this to get ref
```

or

```js
<input {...getFieldProps('ref',{
  ref: this.saveRef // use function here or use getFieldDecorator to write inside <input> (only allow function)
})} />
```

## Test Case

```
npm test
npm run chrome-test
```

## Coverage

```
npm run coverage
```

open coverage/ dir

## License

rc-form is released under the MIT license.
