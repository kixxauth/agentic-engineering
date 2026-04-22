In the HyperviewService defined in lib/hyperview/hyperview-service.js we have an initialize method which takes these parameters:

```javascript
/**
 * @param {Object} options - Service initialization options
 * @param {Object} options.pageStore - PageStore instance for loading page data and templates
 * @param {Object} options.templateStore - TemplateStore instance for loading base templates and partials
 * @param {Object} options.templateEngine - TemplateEngine instance for compiling templates
 * @param {Object} options.staticFileServerStore - StaticFileServerStore instance for serving static files
 */
```

We need to refactor the HyperviewService class so that the pageStore, templateStore, templateEngine, and staticFileServerStore are passed into the constructor and then initialize() can be called without arguments.
