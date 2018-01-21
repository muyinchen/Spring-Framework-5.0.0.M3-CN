## 19.12 JSON Mapping View

The`MappingJackson2JsonView`uses the Jackson library’s`ObjectMapper`to render the response content as JSON. By default, the entire contents of the model map \(with the exception of framework-specific classes\) will be encoded as JSON. For cases where the contents of the map need to be filtered, users may specify a specific set of model attributes to encode via the`RenderedAttributes`property. The`extractValueFromSingleKeyModel`property may also be used to have the value in single-key models extracted and serialized directly rather than as a map of model attributes.

JSON mapping can be customized as needed through the use of Jackson’s provided annotations. When further control is needed, a custom`ObjectMapper`can be injected through the`ObjectMapper`property for cases where custom JSON serializers/deserializers need to be provided for specific types.

[JSONP](https://en.wikipedia.org/wiki/JSONP)is supported and automatically enabled when the request has a query parameter named`jsonp`or`callback`. The JSONP query parameter name\(s\) could be customized through the`jsonpParameterNames`property.

