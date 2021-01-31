---
id: quickstart
title: Quickstart
sidebar_label: Quickstart
---

## Install

```bash
npm i @formx/formy
```

## Create your form class

```ts
import { IForm, Field } from "@formx/formy";

class PersonForm implements IForm {
  firstName = Field(TextField);
  lastName = Field(TextField);
  fullName = Field(TextField);

  init = context => {
    this.firstName.value = context.person?.firstName ?? "";
    this.lastName.value = context.person?.lastName ?? "";
    this.fullName.value = context.person
      ? `${context.person.firstName} ${context.person.lastName}`
      : "";
    this.firstName.schema = yup.string().required();
  };

  update = (context, reason) => {
    this.firstName.props = {
      label: "First name",
    };
    this.lastName.props = {
      label: "Last name",
    };
    this.fullName.props = {
      label: "Full name",
      isDisabled: true,
    };
    this.calculateFullName(reason);
  };

  calculateFullName = (reason: UpdateReason) => {
    if (
      reason.type === "value" &&
      (reason.name === "firstName" || reason.name === "lastName")
    ) {
      this.fullName.value = `${this.firstName.value} ${this.lastName.value}`;
    }
  };
}
```

## Create your component

```tsx
const TextField = ({ label, isVisible, field, background }) => {
  if (typeof isVisible === "boolean" && !isVisible) return null;

  return (
    <div style={{ background }}>
      <h2>{label}</h2>
      <input
        type="text"
        {...field}
        value={field.value ?? ""}
        onChange={e => field.onChange(e.target.value)}
        ref={field.focusRef}
      />
      {field.error}
    </div>
  );
};
```

### Explanation

There is a special prop passed to you components called `field`.
It includes following things:

- value
- onChange
- onBlur
- error
- focusRef (should be passed to something with a`focus()` method.

Every other prop is arbitrary.

These props are passed with `this.fieldName.props = {}` in the class.

## Wire everything up

```tsx
import { Form } from "@formx/formy";

const App = () => {
  const [person, setPerson] = useState < any > null;
  const formRef = useRef < Form > null;

  useEffect(() => {
    (async () => {
      const person = await getPerson();
      setPerson(person);
    })();
  }, []);

  useEffect(() => {
    if (person) {
      if (!formRef.current) return;
      formRef.current.resetForm();
    }
  }, [person]);

  const onSubmit = (values, transformedValues, args) => {
    console.log(values, transformedValues);
  };

  const context = {
    person,
  };

  return (
    <div>
      <Form
        form={PersonForm}
        onSubmit={onSubmit}
        context={context}
        ref={formRef}
      >
        {({ fields, submitForm, resetForm, focusField }) => {
          return (
            <>
              {fields.firstName}
              {fields.lastName}
              {fields.fullName}
              <br />
              <br />
              <button onClick={() => submitForm()}>Submit</button>
              {/* mode will be in the args object passed to onSubmit */}
              <button onClick={() => submitForm({ mode: "special" })}>
                Special submit
              </button>
              <button onClick={() => resetForm()}>Reset</button>
              <button onClick={() => focusField("firstName")}>
                Focus first name
              </button>
            </>
          );
        }}
      </Form>
    </div>
  );
};
```

### Explanation

We're using the Form component and we're passing it 4 things:

- `form` - Your form class
- `onSubmit` - Function that will be called when you submit if all fields are valid. It will be called with values and transformed values.
- `context` - External dependencies. In this case, it's a person object that we get from the server.
- `ref`- Ref to the Form component. We use this to reset the form from the outside when the person object arrives from the server.

## Using form context

```tsx
const SomeNestedComponent = () => {
  const { getFieldsStack, submitForm } = useFormContext();

  return (
    <>
      {getFieldsStack()}
      <button onClick={() => submitForm()}>Submit</button>
    </>
  );
};
```

### Explanation

`useFormContext` returns all the same things you get in the child function of the `Form` component (see above).

## Using subscription

```tsx
const SomeNestedComponent = () => {
  const { values, errors, previousValues, previousErrors } = useSubscribe();

  return (
    <>
      List of all errors
      {Object.values(errors).map(err => {
        return <div>{err}</div>;
      })}
    </>
  );
};
```

### Explanation

`useSubscribe` will update your component each time it's return values change.
