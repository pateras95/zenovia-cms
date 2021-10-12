<template>
  <div class="home">
    nothing
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    name: 'Home',
    data() {
      return {
        testArray: []
      }
    },
    created() {
      axios.get('http://localhost:8080/static/test.xml', {
          headers: {
            Accept: 'application/xml'
          }
        })
        .then(response => {
          // console.log(response.data)
          var XmlNode = new DOMParser().parseFromString(response.data, 'text/xml');
          this.xmlToJson(XmlNode)
          console.log('Name :' , this.testArray[7]['@attributes'].Name)
        })
    },
    methods: {
      xmlToJson(xml) {
        // Create the return object
        var obj = {};

        if (xml.nodeType == 1) {
          // element
          // do attributes
          if (xml.attributes.length > 0) {
            obj["@attributes"] = {};
            for (var j = 0; j < xml.attributes.length; j++) {
              var attribute = xml.attributes.item(j);
              obj["@attributes"][attribute.nodeName] = attribute.nodeValue;
            }
          }
        } else if (xml.nodeType == 3) {
          // text
          obj = xml.nodeValue;
        }

        // do children
        // If all text nodes inside, get concatenated text from them.
        var textNodes = [].slice.call(xml.childNodes).filter(function (node) {
          return node.nodeType === 3;
        });
        if (xml.hasChildNodes() && xml.childNodes.length === textNodes.length) {
          obj = [].slice.call(xml.childNodes).reduce(function (text, node) {
            return text + node.nodeValue;
          }, "");
        } else if (xml.hasChildNodes()) {
          for (var i = 0; i < xml.childNodes.length; i++) {
            var item = xml.childNodes.item(i);
            var nodeName = item.nodeName;
            if (typeof obj[nodeName] == "undefined") {
              obj[nodeName] = this.xmlToJson(item);
            } else {
              if (typeof obj[nodeName].push == "undefined") {
                var old = obj[nodeName];
                obj[nodeName] = [];
                obj[nodeName].push(old);
              }
              obj[nodeName].push(this.xmlToJson(item));
            }
          }
        }
        this.testArray.push(obj)
      }
    }
  }
</script>