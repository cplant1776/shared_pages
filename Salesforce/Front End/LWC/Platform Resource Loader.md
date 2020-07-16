Platform Resource Loader

# [Overview](https://developer.salesforce.com/docs/component-library/bundle/lightning-platform-resource-loader/documentation)

Used to import 3rd party JS and CSS files

## Use

1. Upload as a static resource
2. Import the resource in your controller
3. Call **loadScript(self, fileUrl)**, which is a Promise

## Example

```javascript
import { LightningElement } from 'lwc';
import { loadStyle, loadScript } from 'lightning/platformResourceLoader';

import leaflet from '@salesforce/resourceUrl/leaflet'

export default class MyMap extends LightningElement {
    // invoke the loaders in connectedCallback() to ensure that
    // the page loads and renders the container before the map is created

  connectedCallback() {

    Promise.all([
      loadStyle(this, leaflet + '/leaflet.css'),
      loadScript(this, leaflet + '/leaflet.js')
    ]).then(() => {

// initialize the library using a reference to the container element obtained from the DOM
      const el = this.template.querySelector('div');
      const mymap = L.map(el).setView([51.505, -0.09], 13);

// Load and display tile layers with your access token
    L.tileLayer('https://api.tiles.mapbox.com/v4/{id}/{z}/{x}/{y}.png?access_token={accessToken}', {
          maxZoom: 18,
          id: 'mapbox.streets',
          accessToken: 'your.mapbox.access.token'
      }).addTo(mymap);
    });
  }
}
```