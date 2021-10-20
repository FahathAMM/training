# Building A Support System - Part 4

This is the third part of a series of tutorials guiding to develop an **Online Support System**.
- [Part 1](./building-a-support-system-1.md)
- [Part 2](./building-a-support-system-2.md)
- [Part 3](./building-a-support-system-3.md)

In **Part 4** we focus on integrating some JavaScript and adding more features to the support system.





## File Uploads and Storage


```html
<!-- FORM START -->
<form class="" action="{{ route('tickets.store') }}" method="post">
    {{ csrf_field() }}

    <div class="row justify-content-center">
        <div class="col-lg-6">

            <div class="form-group row">
                <div class="col-md-4 text-md-right">
                    <label for="customer_name">Your Name</label>
                </div>
                <div class="col-md-8">
                    <input type="text" name="customer_name" value="" class="form-control">
                </div>
            </div>

            <div class="form-group row">
                <div class="col-md-4 text-md-right">
                    <label for="email">Email</label>
                </div>
                <div class="col-md-8">
                    <input type="text" name="email" value="" class="form-control">
                </div>
            </div>

            <div class="form-group row">
                <div class="col-md-4 text-md-right">
                    <label for="phone">Phone</label>
                </div>
                <div class="col-md-8">
                    <input type="text" name="phone" value="" class="form-control">
                </div>
            </div>

            <div class="form-group row">
                <div class="col-md-4 text-md-right">
                    <label for="description">Description</label>
                </div>
                <div class="col-md-8">
                    <textarea name="description" class="form-control"></textarea>
                </div>
            </div>

            <div class="form-group row">
                <div class="col-md-4 text-md-right">
                    <label for="attachment">Attachment</label>
                </div>
                <div class="col-md-8">
                    <input type="file" name="attachment" value="">
                    <div class="text-muted">
                      Max upload size: 2MB. Only the file with jpg, png, gif and pdf extensions are valid.
                    </div>
                </div>
            </div>

            <div class="row">
                <div class="col-md-8 offset-md-4 text-md-right">
                    <input type="submit" value="Submit" class="btn btn-success">
                </div>
            </div>
        </div>
    </div>
</form>
<!-- FORM END -->
```

## AJAX

It is straightforward to submit the forms using HTML. We used such form when adding a comment to a ticket. However, each time you add a comment, the ticket view is reloaded. It can be annoying at some point, where you are at the bottom of the page and the browser jumps to the top of the page after adding a new comment.  A little JavaScript touch can improve the user experience a lot. Instead of submitting the form using HTML, you can use **AJAX** to submit the form. Let's see how we can do that.

We use jQuery in the above code.

- `$('#comment-form')` selects the `<form>` element with the ID `comment-form` in the ticket view. Learn more about [jQuery selectors](https://api.jquery.com/category/selectors/) to understand how to select DOM elements for manipulations.
- `on()` function binds a handler to the `submit` event of the form.
- `$evt.preventDefault()` prevents the default behavior of the submit event allowing the rest of the code in the handler function to take care of the submission.
`$.ajax()` function makes a **HTTPRequest** call to the server using the URL in `action` attribute of the form. It also serializes and passes the form data to the request.
