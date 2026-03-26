
[Stage 1](https://github.com/dr-matt-smith/FEDev-form-validation-with-user-error-messages---stage-1)
|
[Stage2](https://github.com/dr-matt-smith/FEDev-form-validation-with-user-error-messages--stage-2)
|
[Stage3](https://github.com/dr-matt-smith/FEDev-form-validation-with-user-error-messages-stage-3)
|
Stage 4


# FEDev-form-validation-with-user-error-messages


# stage 4 - show the user the form validation errors 

Let's pass on the error messages we have caught back up to the web page from the form processing script ...

## use `fail()`  function to return an error message from server script to the Svelte form

We're going to use the `fail()` so we need to import this from core SvelteKit resources.

Add this `import` statement at the beginning on `/routes/+page.server.js`:

```javascript
import { fail } from '@sveltejs/kit';
```

Now as well as logging when an error is caught, we'll also call this `fail()` function
- we'll send both the attempted TODO `description`, and also the error 'message`


```javascript
    ... code before

        } catch (error) {
            let message = error.message;
    
            // LOG caught ERROR
            console.log("TODO - *** ERROR *** - there was a problem with your new TODO = '" +  + message + "'");
    
            return fail(422, {
                description: description,
                error: message
            });
        }

    ... code after
```

So the complete listing now looks as follows:

`/routes/+page.server.js`

```javascript
    import { fail } from '@sveltejs/kit';
    import * as db from '$lib/server/database.js';
    
    export function load({ cookies }) {
        let id = cookies.get('userid');
    
        if (!id) {
            id = crypto.randomUUID();
            cookies.set('userid', id, { path: '/' });
        }
    
        return {
            todos: db.getTodos(id)
        };
    }
    
    export const actions = {
        create: async ({cookies, request}) => {
            const data = await request.formData();
            let description = data.get('description');

            try {
                db.createTodo(cookies.get('userid'), description);
    
                // LOG successful TODO creation
                console.log("TODO success! data valid, created new TODO = '" + description + "'");
    
            } catch (error) {
                let message = error.message;
    
                // LOG caught ERROR
                console.log("TODO - *** ERROR *** - there was a problem with your new TODO = '" + message + "'");
    
                return fail(422, {
                    description: description,
                    error: message
                });
            }
        },
    
        delete: async ({ cookies, request }) => {
            const data = await request.formData();
            db.deleteTodo(cookies.get('userid'), data.get('id'));
        }
    };
```


## receive and display message about errors to user in Svelte form

First, since the form processing script may be sending out page data, we need to get our props needs to come from both:
- `data` (the server script `load()` function)
- and also `form` (the above server script `actions`)

So at the beginning of `/routes/+page.svelte` update our proops import statement as follows:

```javascript
    <script>
        let { data, form } = $props();
    </script>
```

Now we can display any error message in `form`:

```javascript
    {#if form?.error}
        <p class="error">{form.error}</p>
    {/if}

    ...this

    <style>
        ...
        
        .error {
            padding: 1rem;
            background-color: lightpink;
        }
    </style>
```

NOTE:
- `form?.error` is a short way of writing:
  - IF `form` is not underfined and `form` contains a property `error`

Now the user sees a clear message about what went wrong!

![](/screenshots/50_web_error_message.png)