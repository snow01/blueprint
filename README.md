blueprint
=========

An Advanced Template System for HTML. 

+ Blueprint templates are html - no other language to learn. 
+ Blueprint templates are recursive in nature - a template can include another template, that can include other templates, and so on.
+ Blueprint templates are closure functions - can call other templates, can pass-in variables.
+ Blueprint templates give semantic structure to html - define meaningful names to reusable template blocks (such as separator, thumbnail list, empty list)

Building complex html from re-usable templates
----------------------------------------------

Let's take following complex html, as an example. I will demonstrate step-by-step method to convert this html to use blueprint templating.

In my application, I had to show lists of items. I have various types of items - enterprise category, enterprise, group etc. In this list page, I have a page-header that tells what list it is. Page-header includes link to create more items of the same type. I can have an empty list or list with items. In case of empty list, I wanted to show a different message that prompts user about no item and gives action button to create a new item. In case of list with items, I show the items in a thumbnail layout. Each thumbnail is clickable and click leads to item details.

If you would have noticed, this layout is pretty complex and this layout is similar for other types of items - for another type of item, things that differ are - (a) list variable name (b) name of the list for display (c) content of the item.

```html
  <div> 
   <!-- PAGE-HEADER -->
   <div class="row"> 
    <h2 class="page-header"><span class="glyphicon glyphicon-th"></span>Enterprise Categories List<a class="pull-right" href="#/enterpriseCategories/create" ng-if="enterpriseCategories.length > 0"><span class="glyphicon glyphicon-plus">
       <!-- icon --></span></a></h2> 
    <div class="clearfix empty-separator">
     <!-- clearfix -->
    </div> 
   </div> 
   
   <!-- WHEN LIST IS EMPTY -->
   <div class="well text-center" ng-cloak="" ng-if="!enterpriseCategories || enterpriseCategories.length == 0"> 
    <div class="row lead">
     No enterprise category exists. Start by creating one
    </div> 
    <div class="clearfix empty-separator">
     <!-- clearfix -->
    </div> 
    <div class="row"> 
     <a class="btn btn-lg btn-primary" href="#/enterpriseCategories/create" role="button">Create</a> 
    </div> 
    <div class="clearfix empty-separator">
     <!-- clearfix -->
    </div> 
   </div> 
   
   <!-- WHEN LIST HAS ITEMS -->
   <div ng-cloak="" ng-if="enterpriseCategories.length > 0"> 
    <div class="row thumbnail-container" equalize=".thumbnail"> 
     <div class="col-xs-12 col-sm-6 col-md-4 col-lg-3" ng-repeat="item in enterpriseCategories"> 
      <div class="thumbnail"> 
       <a href="/#/enterpriseCategories/{{item.id}}">  
        <div class="caption text-center"> 
         <h4 class="title">{{item.name}}</h4> 
         <h4 class="description">{{item.description}}</h4> 
         <h5 class="stats badge">{{schemaCount}} schemas</h5>
         <h5 class="stats badge">{{enterpriseCount}} enterprises</h5>
         <h5 class="stats badge">{{userCount}} users</h5> 
        </div> </a> 
      </div> 
     </div> 
    </div> 
   </div> 
  </div>
```

### Step 1

Simplest thing first - if you notice above html has repeated instance of following block -

```html
<div class="clearfix empty-separator">
  <!-- clearfix -->
</div>
```

I defined a 'separator.html' template file -

```html
<div id="separator" class="clearfix empty-separator"><!-- clearfix --></div>
```

id of the node defines the name of the template. I can use this template block in following way -

```html
<separator/>
```

### Step 2

For the page-header, I defined 'page-header.html' template file -

```html
<div id="page-header" class="row">
  <h2 id="heading" class="page-header">
    <span class="glyphicon glyphicon-th"></span> Some Heading 
    <a id="create-link" class="pull-right" href="/some-link.html">
      <span class="glyphicon glyphicon-plus"><!-- icon --></span>
    </a>
  </h2>
  <separator/>
</div>

```

This is bit complex template than the separator template. 

1. Id 'page-header' of the root node defines name of the template.

2. This template has two sub-templates (templates that can only be used within main template) - heading, and create-link.

3. This template re-uses separator template.

Here is how I use this template in the view file -

```html
<page-header>
  <heading>Enterprise Category List</heading>
  <create-link href="#/enterpriseCategories/create" ng-if="enterpriseCategories.length > 0"/>
</page-header>
```
Few things to note here -

1. This usage of page-header has two sub-nodes heading and create-link

2. heading node has the text - by just defining the text, engine replaces "Some Heading" in template with this text value - absolutely no additional syntax or language constructs is needed. ==> Natural text templating

3. create-link node defines two attributes: href and ng-if. use of href attribute in view, overrides the value "/some-link.html" with value "#/enterpriseCategories/create". ==> Natural attribute override

4. ng-if is a new attribute, that was not part of the template. By defining any new attribute - it automatically gets added in the right note. ==> Natural attribute addition

Output html looks following - 

```html
<div class="row"> 
  <h2 class="page-header"><span class="glyphicon glyphicon-th"></span>Enterprise Categories List<a class="pull-right" href="#/enterpriseCategories/create" ng-if="enterpriseCategories.length > 0"><span class="glyphicon glyphicon-plus">
       <!-- icon --></span></a></h2> 
  <div class="clearfix empty-separator">
     <!-- clearfix -->
  </div> 
</div> 
```

### Step 3

Similarly for empty-list block I defined following template block in 'empty-list.html' file - 

```html
<div id="empty-list" class="well text-center" ng-cloak="">
    <div id="message" class="row lead">No list-name exists. Start by creating one</div>
    <separator/>
    <div class="row">
        <a id="create-link" class="btn btn-lg btn-primary" href="/some-link.html"
           role="button">Create</a>
    </div>
    <separator/>
</div>
```

I use this template in view file as follows -

```html
<empty-list ng-if="!enterpriseCategories || enterpriseCategories.length == 0" >
  <message>No enterprise category exists. Start by creating one</message>
  <create-link href="#/enterpriseCategories/create"/>
</empty-list>
```

### Step 4

For list with items, I define following template block in 'thumbnail-list.html' file -

```html
<div id="thumbnail-list" ng-cloak="">
    <div class="row thumbnail-container" equalize=".thumbnail">
        <div id="list" class="col-xs-12 col-sm-6 col-md-4 col-lg-3" ng-repeat="item in list">
            <div class="thumbnail">
                <a id="item" href="/some-link.html">
                  <img id="icon" ng-src="{{item.icon}}" width="128" height="128">

                  <div class="caption text-center">
                    <h4 id="title" class="title">{{item.name}}</h4>
                    <h4 id="description" class="description">{{item.description}}</h4>
                    <h5 id="stats" class="stats badge">{{schemaCount}} schemas</h5>
                  </div>
                </a>
            </div>
        </div>
    </div>
</div>
```

Here is how I use it in the view file -

```html
<thumbnail-list ng-if="enterpriseCategories.length > 0">
  <list ng-repeat="item in enterpriseCategories">
    <item>
      <title>{{item.name}}</title>
      <description>{{item.description}}</description>
      <stats>{{schemaCount}} schemas</stats>
      <stats>{{enterpriseCount}} enterprises</stats>
      <stats>{{userCount}} users</stats>
    </item>
  </list>
</thumbnail-list>
```

A few important things to note here -

1. As earlier, name of the template is 'thumbnail-list' as root node defines id 'thumbnail-list'

2. Deep down, it defines a sub-template with name 'item', which iternally defines sub-templates 'icon', 'title', 'description', and 'stats'

3. See the usage. 

4. thumbnail-list node has item child node. It also has ng-if as new attribute.

5. item node does not have 'icon' child node ==> icon element would not be included in the output html ==> Natural node removal

6. stats node is repeated 3 times ==> there will be 3 instances of stats element in the output html ==> Natural collections

Output html looks as following - 

```html
<div ng-cloak="" ng-if="enterpriseCategories.length > 0"> 
    <div class="row thumbnail-container" equalize=".thumbnail"> 
     <div class="col-xs-12 col-sm-6 col-md-4 col-lg-3" ng-repeat="item in enterpriseCategories"> 
      <div class="thumbnail"> 
       <a href="/#/enterpriseCategories/{{item.id}}">  
        <div class="caption text-center"> 
         <h4 class="title">{{item.name}}</h4> 
         <h4 class="description">{{item.description}}</h4> 
         <h5 class="stats badge">{{schemaCount}} schemas</h5>
         <h5 class="stats badge">{{enterpriseCount}} enterprises</h5>
         <h5 class="stats badge">{{userCount}} users</h5> 
        </div> </a> 
      </div> 
     </div> 
    </div> 
   </div>
```

### Step 5

Combining these 3 blocks (page-header, empty-list, thumbnail-list), I created a layout template named "list-page.html". Here is how it looks like -

```html
<div id="list-page">
    <page-header >
        <heading>#{list-name-plural} List</heading>
        <create-link href="#{create-link}"
                     ng-if="#{list}.length > 0"/>
    </page-header>
    <empty-list ng-if="!#{list} || #{list}.length == 0" >
        <message>No #{list-name-singular} exists. Start by creating one</message>
        <create-link href="#{create-link}"/>
    </empty-list>
    <thumbnail-list ng-if="#{list}.length > 0">
      <list ng-repeat="item in #{list}"/>
    </thumbnail-list>
</div>
```

Here is how use of this layout template looks like -

```html
<list-page var-list="enterpriseCategories"
           var-list-name-singular="enterprise category"
           var-list-name-plural="Enterprise Categories"
           var-create-link="#/enterpriseCategories/create"
        >
  <item href="/#/enterpriseCategories/{{item.id}}">
    <title>{{item.name}}</title>
    <description>{{item.description}}</description>
    <stats>{{schemaCount}} schemas</stats>
    <stats>{{enterpriseCount}} enterprises</stats>
    <stats>{{userCount}} users</stats>
  </item>
</list-page>
```

A few important things to note in this step -

1. I introduced variables in this step. Variables are defined as #{variable-name}

2. In view file, I defined the variable value by using attribute var-variable-name, eg: var-list, var-create-link, etc.

3. In the layout template file, I did not fully created the view for thumbnail-list by defining item and its content - as in this page I do not know the content. This way, item template resolution becomes responsibility of final view ==> Lazy template resolution.

Few more features that are not demonstrated above
-------------------------------------------------

1. This template engine integrates tightly with spring - it provides view resolver based implementation.

2. Within templates we can define SpEL based expressions, these expressions would be evaluated against spring context.

3. SpEL based expressions also provide following flow constructs -

...a. s-show-if --- include the node if condition is true.

...b. s-hide-if --- remove the node if condition is true.

...c. s-for -- iterate for the collection, sub nodes can have s-odd-class and s-even-class conditions

...d. s-switch & s-switch-on -- switch conditions

...e. s-class-if -- conditional classes

4. Automatic internationalization support, with resolution against Messages resource. No need to write cumbersome spring expressions to have internationalization support. You can simply define string resources for <view-name>.<dot-separated-semantic-path-to-attribute or element-text> - during rendering, texts will be resolved from message resources.

5. A single file can have multiple named templates.


Upcoming features
-----------------

1. Maven and gradle integration tasks to generate keys for internationalization resource.

2. Eclipse and IntelliJ integration for auto-completion

3. Javascript based engine for client side template expansion.

4. Support for head section templates.

Thanks !
