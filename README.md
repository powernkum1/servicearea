# TWA Service Area — updated_servicearea.html

## Overview

This repository contains a single HTML file, `updated_servicearea.html`, which implements an interactive map-based service-area viewer for TWA (Toho Water Authority). The app uses the ArcGIS JavaScript API (v4.31) to display service-area polygons and to provide tools for searching, reverse geocoding, feature inspection, and user-friendly popups.

The README documents the file structure, functionality, configuration options, runtime requirements, troubleshooting guidance, and notes for future improvements.

## Files

- `updated_servicearea.html` — The main (and only) application file. It:
  - Loads ArcGIS JS API v4.31 via CDN.
  - Adds a `FeatureLayer` for service-area polygons and additional FeatureLayers for water, sewer, and irrigation/reclaim service areas.
  - Implements click handlers to determine whether a point is inside the service area and shows a popup accordingly.
  - Performs reverse geocoding using the ArcGIS World Geocoding service (`geocode.arcgis.com`).
  - Provides a Search widget, Home widget, Basemap toggle, and a custom Legend UI.
  - Adds a draggable popup implementation and a custom floating div when users click outside the service area.
  - Uses a geometry buffer (200 feet) to detect locations close to water boundaries and provide a cautionary message.

## Key Features

- Basemap: `streets-navigation-vector` with a toggle to `satellite`.
- Service area display: main `FeatureLayer` URL is configured in the file and used in the Legend.
- Search: ArcGIS `Search` widget with customized placeholder and results handling. Search results are tested against the service-area layer to determine whether they're inside or outside the service area.
- Reverse geocoding: Uses ArcGIS geocode service to turn map points into human-readable addresses.
- Click handling:
  - If the user clicks on the service-area feature, a popup is shown indicating the address and the available services (water, sewer, reclaim irrigation).
  - If the clicked point is outside the service area, a floating custom div appears with the address and a Close button.
- Service availability logic: Intersects the clicked or searched point with three FeatureLayers (water, sewer, irrigation) to return a natural-language message describing available services.
- Near-boundary detection: If the point is not inside the water area but falls within a 200-foot buffer of the water boundary, a 'near boundary' message is shown with contact details.
- Contact details included in the UI messages: customerservice@tohowater.com and phone 407-944-5000.
- Draggable popups: Custom drag handling on the popup main container so the popup can be moved by the user.
- Legend toggle: Custom HTML container for the Legend, with animated hide/show.

## Important Configuration Values

Within `updated_servicearea.html`, the important values you'll likely modify are:

- Feature layer URLs:
  - Main service area layer:
    - URL used: `https://services3.arcgis.com/PPOHFcbrL50ceHvX/arcgis/rest/services/Servicearea_shapefile/FeatureServer/0`
  - Service availability check base (layers):
    - `https://services3.arcgis.com/PPOHFcbrL50ceHvX/arcgis/rest/services/Water_Reclaim_Sewer_ServiceAreas/FeatureServer/0` — irrigation
    - `.../1` — sewer
    - `.../2` — water
  If you need to switch to other layers or another server, update the URLs used when creating `FeatureLayer` instances.

- ArcGIS API version: loaded from `https://js.arcgis.com/4.31/` in the HTML head. If you want a different version, update the script and theme link accordingly.

- Reverse geocoder: currently set to `https://geocode.arcgis.com/arcgis/rest/services/World/GeocodeServer`. If you have an enterprise geocoder or require an API key/token, update the `locator` calls accordingly and add the required authentication.

- Buffer distance and units: The buffer for near-boundary detection is currently `200` (feet). You can change the distance and unit inside the `checkServicesAtPoint` function if needed.

## How It Works (High-Level)

1. Page loads and initializes an ArcGIS `Map` and `MapView`.
2. The main service-area `FeatureLayer` is added to the map.
3. Widgets are added: Search, Home, BasemapToggle, Legend.
4. Click events: The map view listens for `click` events. A `hitTest` is used to check whether the click intersected any feature from the service-area layer.
   - If a service-area feature is clicked, `checkServicesAtPoint` queries the water/sewer/irrigation layers and constructs a human-readable availability message.
   - If no feature is hit, reverse geocoding is run and a small floating div displays the address and a message that the location is outside the service area.
5. Search results are handled similarly: results are reverse-geocoded and tested against the service-area layer. The popup content is adjusted based on whether the location intersects a service-area feature.

## Key Functions and Variables (in-file reference)

- `featureLayer` — main service-area FeatureLayer instance (popup intentionally disabled to allow custom behavior).
- `checkServicesAtPoint(mapPoint)` — main helper function that checks three service-related feature layers for intersections, builds a readable list (e.g., "water and sewer"), and returns a formatted string message. Also performs a buffer-based near-boundary check when the point is not in the water layer.
- `addressOnFeature` — a module-level variable used to store the reverse-geocoded address for use in popups and custom divs.

## How to Run Locally

This is a client-side HTML app. For full functionality, serve the file from a web server (ArcGIS JS API has CORS and loading behaviors that work best via HTTP(S)). Two simple options:

- Using Node (if you have npm):

```bash
# in PowerShell or WSL/bash (from this repo folder)
# install 'serve' once (optional):
npm install -g serve
# then serve the current folder:
serve .
```

- Python 3 built-in server:

```bash
# from the directory containing updated_servicearea.html
python -m http.server 8000
# then open http://localhost:8000/updated_servicearea.html
```

Directly opening the file in a browser (`file://`) may work in some browsers but can lead to CORS or resource-loading issues — a local HTTP server is recommended.

## Dependencies & Network Requirements

- Internet connectivity required to load ArcGIS JS API from `https://js.arcgis.com/4.31/` and to access ArcGIS REST services used for FeatureLayers and geocoding.
- If your environment requires API keys or tokens for ArcGIS services, you'll need to wire the token or OAuth authentication into the app. The current code assumes publicly accessible REST endpoints.

## Troubleshooting & Notes

- Blank map or missing layers:
  - Confirm that `https://js.arcgis.com/4.31/` is reachable from your network.
  - Check browser console for CORS or 403/401 errors when trying to access the FeatureLayer URLs.

- Reverse geocoding failing or returning limited results:
  - The app calls ArcGIS's public geocoding endpoint. If your usage is high or rate-limited, consider provisioning your own geocode service or add an API key/token if required.

- Popups not draggable or DOM errors:
  - The app uses a MutationObserver and direct DOM manipulations. If the ArcGIS API internals change in future versions, the code may require adjustments.

- Near-boundary warnings:
  - The app uses a 200-foot buffer to detect "near boundary" cases. This is an application-specific choice and can be tuned for your needs.

- Accessibility:
  - The UI uses standard HTML controls. Further improvements may include ARIA labels for custom elements, keyboard accessibility for popups, and better color contrasts for visibility.

## Testing & Validation

Manual tests you can run:
- Click inside a clearly visible service area polygon — the popup should say "Within Service Area" and list available services.
- Click outside the service area — a floating div should appear with the reverse-geocoded address (if available) and a notice that the location is outside the service area.
- Use Search to find an address inside and outside service areas and verify results and popups behave as expected.
- Click very near the water boundary (within ~200 ft) to confirm the "near our boundary" message appears.

Automated tests are not provided in this static HTML, but you can script basic end-to-end checks using headless browsers or Puppeteer to assert that popups open and content includes expected strings.

## Security & Privacy

- No user data is stored by this static example. Reverse geocoding requests are sent to ArcGIS servers.
- If you add analytics or logging, ensure that personal/sensitive data handling complies with applicable privacy rules.

## Future Improvements (suggested)

- Add configuration file (JSON) for layer URLs and buffer distance so runtime changes don't require editing HTML.
- Add unit or integration tests (Puppeteer-based) for automated verification of popups, search, and click behavior.
- Extract inline styles into a separate CSS file for better maintainability.
- Add optional token-based authentication for ArcGIS services requiring an API key / token.
- Improve accessibility (ARIA roles, keyboard support) and responsive layout handling.

