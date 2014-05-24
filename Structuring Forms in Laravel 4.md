So... building forms in Laravel 4. 
Over the past year I've practiced a lot, and made countless small, useless, never-to-see-the-light-of-day projects in Laravel, to get better at structuring applications. One of the things that have annoyed me the most is creating form templates, Laravel have a great FormBuilder class that makes it very easy to build a basic form fast.

For example an inputbox and label for a title:

´´´´
{{ Form::label('title', 'Title:') }}
{{ Form::text('title', null) }}
´´´´

But when there is need for a little more markup, classes, placeholders, and most important when you need a lot of form elements. Then it gets a little "clunky" and repetitative, for example:

´´´´
<!-- Text -->
<div class="form-group {{ $errors->has('title') ? 'has-error' : '' }}">
    {{ Form::label('title', 'Title:', ['class' => 'col-sm-2 col-md-2 control-label']) }}
    <div class="col-sm-10 col-md-10">
        {{ Form::text('title', null, array('class' => 'form-control', 'placeholder' => 'Write your title here')) }}
        {{ $errors->has('title') ? $errors->first('title', '<span class="help-block">:message</span>'): '' }}
    </div>
</div>
<!-- Text -->
<div class="form-group {{ $errors->has('slug') ? 'has-error' : '' }}">
    {{ Form::label('slug', 'Slug:', ['class' => 'col-sm-2 col-md-2 control-label']) }}
    <div class="col-sm-10 col-md-10">
        {{ Form::text('slug', null, array('class' => 'form-control', 'placeholder' => 'Write your slug here')) }}
        {{ $errors->has('slug') ? $errors->first('slug', '<span class="help-block">:message</span>'): '' }}
    </div>
</div>
<!-- Select -->
<div class="form-group {{ $errors->has('category_id') ? 'has-error' : '' }}">
    {{ Form::label('category_id', 'Category:', ['class' => 'col-sm-2 col-md-2 control-label']) }}
    <div class="col-sm-10 col-md-10">
        {{ Form::select('category_id', $listOfCategories, null, array('class' => 'form-control')) }}
        {{ $errors->has('category_id') ? $errors->first('category_id', '<span class="help-block">:message</span>'): '' }}
    </div>
</div>
<!-- Textarea -->
<div class="form-group {{ $errors->has('body') ? 'has-error' : '' }}">
    {{ Form::label('body', 'Body:', ['class' => 'col-sm-2 col-md-2 control-label']) }}
    <div class="col-sm-10 col-md-10">
        {{ Form::textarea('body', null, array('class' => 'form-control',  'placeholder' => 'Your bodytext goes here', 'rows' => 8)) }}
        {{ $errors->has('body') ? $errors->first('body', '<span class="help-block">:message</span>'): '' }}
    </div>
</div>
´´´´

Now imagine you have a lot of different forms using the same markup, and you then need to change the markup for the text input "block"... not funny.

## Solution 1: Extending FormBuilder

I already saw a great solution to this problem in a video on Laracasts: [Form Macros for the Win](https://laracasts.com/lessons/form-macros-for-the-win). You need a subscription to see the video (which I would recommend you to get anyway, you won't regret it ;) ), but you can see the code here: [FormBuilder.php](https://github.com/laracasts/Laravel-Form-Macros-for-the-Win/blob/master/app/Acme/Html/FormBuilder.php). What he does is extending Laravels own FormBuilder and generating the field "blocks". And ends up being able to use this in his views:

´´´´
{{ Form::textField('username', 'Username:') }}
{{ Form::textField('email', 'Email:') }}
{{ Form::textField('age', 'Age:') }}
´´´´

This seems very good and DRY, but I still see it as a problem if you would need to change any of the markup. For some cases I would definetly go with this approach, but probably not in most cases.

## Solution 2: Using views

My solution certainly isn't anything magic or fancy, but I haven't seen it anywhere else. So I might as well share it ;)

Instead of mixing the html markup into php classes, I like to keep it in templates. To me it makes more sense to be able to see and change the markup for formelements as html, like the rest of the templates.

I've created a partial view for each form element, so in my views folder I have this:

* form
    - checkbox.blade.php
    - select.blade.php
    - text.blade.php
    - textarea.blade.php

Not going to show all the files here, but here is my `text.blade.php`:

´´´´
{{--
@param  string  $name
@param  string  $label
@param  string  $placeholder
--}}

<div class="form-group {{ $errors->has($name) ? 'has-error' : '' }}">
    {{ Form::label($name, $label, ['class' => 'col-sm-2 col-md-2 control-label']) }}
    <div class="col-sm-10 col-md-10">
        {{ Form::text($name, null, array('class' => 'form-control', 'placeholder' => $placeholder)) }}
        {{ $errors->has($name) ? $errors->first($name, '<span class="help-block">:message</span>'): '' }}
    </div>
</div>
´´´´

The comment at the start of the file is just to show what data you need to pass when including the view.  
And then I have my messy form element with classes, placeholders, and handeling error styling.

Now I can easily build a form that is easy to understand, and reuse the elements to avoid duplicating code:

´´´´
@include('form.text', [
    'name'        => 'title',
    'label'       => 'Title:*',
    'placeholder' => 'The title',
])

@include('form.text', [
    'name'        => 'slug',
    'label'       => 'Slug:',
    'placeholder' => 'The slug',
])

@include('form.select', [
    'name'  => 'category_id',
    'label' => 'Category ID:',
    'list'  => $categoryList,
])

@include('form.textarea', [
    'name'        => 'body',
    'label'       => 'Body:*',
    'placeholder' => 'Your blog post',
    'rows'        => 15,
])
´´´´

I'm not really the big writer, but this was something that worked for me, so I thought it could maybe help someone else :)

And thanks to my lovely [girlfriend](http://camillagejl.com) for proof-reading this, she's the writer, I'm the coder ;)
