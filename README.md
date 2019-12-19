# Motivation
Idle Component does not seek out to be your next UI framework. It does not compete with Bootstrap or Material UI. Instead, if you already have a web application running on Wordpress, Drupal or just static HTML, Idle Component is a great way to add Vue components to your website.

Let's take Wordpress as a great example. Imagine you are tasked with adding a Form on your Wordpress page. You can either:
1. Find a plugin or write PHP code, but it's not as flexible as Vue components
2. Integrate Wordpress with Vue, by using webpack, babel, eslint and all the troubles that come with it. And you must also set up CI/CD, build steps, chunk splitting, bundle size optimization ...
3. Use jQuery plugins to generate forms and validations, but jQuery components are usually of low quality and maintainability 

This dilemma is solved with Idle Components. It makes Frontend simple again: **adding a script tag and start coding right away**. You need a way to define forms? Idle Component gives you a generic &lt;form&gt; component, where you can write your own markup and CSS, and can reuse our existing logic for handling form submit and validation.

In other words, Idle Component is a repository of *Smart* components, components that let you write your own markup and CSS to suit your business needs, while abstracting away some of the logic behind.

The general motto of Idle Component is:

- No npm, webpack, vue-cli, create-react-app...
- No build step, just simple script tag and write your own markup
- Non-intrusive: you can flexibly defines your own HTML and CSS
- Ready-to-use: we try to provide a lot of general-purpose (like &lt;form&gt; or &lt;modal&gt;) and special-purpose (like &lt;wordpress-post-carousel&gt; or &lt;woo-commerce-product-list&gt;)components
- Only business logic: Many components have unit-tested, built-in logic. You just need to implement your own logic.
- Optimized: no need for chunk splitting, each page only loads the components it needs
- Versioned: each component has their own versions number and we will try not to break existing ones.

## Example: Modal
A great example is a Modal component. Although Bootstrap and similar tools already have their modal and it works perfectly fine, you usually needs a whole npm - webpack - vue/react/angular solution. Let's see how you would use &lt;modal&gt; with Idle Component:

```
<html>
<body>
  <template data-idle-component>
    <modal-1.0 init="initModalInstance">
      <template v-slot:header>
        I'm the header
      </template>

      <template v-slot:default>
        <div>
          Hello world, I'm the content now
        </div>
      </template>
    </modal-1.0>
  </template>

  <script src="http://js.idle-component.local/main.min.js"
    data-idle-component
    data-components="['modal-1.0']">
  </script>

  <script>
    function initModalInstance (modalInstance) {
      var loginModal = modalInstance

      setTimeout(function() {
        loginModal.toggleModal()
      }, 10000)
    }
  </script>
</body>
</html>
```

Let's take a look at the script tag. You just need to reference http://js.idle-component.local/main.min.js, and define the components you want, in this case, 'modal-1.0'. After the script executes, you can now freely use <modal-1.0> anywhere in your document. In the second inline script block, we can optionally assign a variable pointing to the &lt;modal-1.0&gt; instance, and we can call methods on it, like toggle the modal after 10 seconds.

## Example: Form

```
<body>
  <template data-idle-component>
    <form-1.0 init="initFormInstance" define-validation="defineValidation">
      <template v-slot:default="{ formData, errors, isLocked }">
        <field-definition-1.0 field="email" value="foo2"></field-definition-1.0>
        <field-definition-1.0 field="password"></field-definition-1.0>

        <input type="text" v-model="formData.email" />
          {{ errors ? errors.email : null }}

        <input type="text" v-model="formData.password" />
          {{ errors ? errors.password : null }}

        <button type="submit" v-if="!isLocked">Submit</button>
        <button type="button" v-else>Please wait...</button>
      </template>
    </form-1.0>
  </template>

  <script src="http://js.idle-component.local/main.min.js"
    data-idle-component
    data-components="['form-1.0', 'field-definition-1.0']">
  </script>

  <script>
    function defineValidation(yup) {
      return yup.object().shape({
        email: yup.string().required(),
        password: yup.string().required()
      })
    }

    function initFormInstance(instance) {
      instance.onSubmit = function(formData, errors) {
        if (!errors) {
          instance.lock()

          setTimeout(() => { // to simulate calling your own backend API
            instance.unlock()
          }, 3000)
        }
      }
    }
  </script>
</body>
```

Similarly, we added the main.min.js script. But this time we need two components: &lt;form-1.0&gt; and &lt;field-definition-1.0&gt;.

The &lt;field-definition-1.0&gt; component will help define which field is present on the form and its default value. Then you will start to write your own &lt;input&gt; markups. If you know a little Vue JS, you can see that we have used v-model to bind inputs to `formData`.

The validation definition is using Yup, a popular Javascript validation library, so no "reinvent the wheel" here and you can reuse your knowledge.

Lastly, in the onSubmit handler, you can get the formData, the simplified errors object and the original Yup ValidationError object. The form instance also has lock()/unlock() method to help you with disabling the button while it's calling remote APIs.
