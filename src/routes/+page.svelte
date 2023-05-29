<script>
  import { onMount } from 'svelte';

  import { currentLocation } from '../stores/geolocation'

  let map;
  let mapContainer;
  let markers = [];
  let currentMarker;

  let name;

  $: mapContainer, setMap();
  $: $currentLocation || markers, updateMarker();

  const setMap = () => {
    if (mapContainer) {
      map = L.map('map-container');
      map.attributionControl.setPrefix('<a href="https://leafletjs.com" title="A JavaScript library for interactive maps">Leaflet</a>');
      // L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
      //   attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
      // }).addTo(map);
      L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Street_Map/MapServer/tile/{z}/{y}/{x}', { attribution: 'Tiles &copy; Esri' }).addTo(map);
      markers = []
      currentMarker = L.marker([$currentLocation.lat, $currentLocation.lon]).addTo(map);
      markers.push(currentMarker)
      map.fitBounds(new L.featureGroup(markers).getBounds());
    }
  }

  const updateMarker = () => {
    if ($currentLocation && currentMarker && $currentLocation.lat && $currentLocation.lon) {
      currentMarker.setLatLng(new L.LatLng($currentLocation.lat, $currentLocation.lon));
      map.fitBounds(new L.featureGroup(markers).getBounds());
    }
  }

  const addPoint = () => {
    const marker = L.marker([$currentLocation.lat, $currentLocation.lon]).bindPopup(name).addTo(map);
    marker.feature = { type: 'Feature', properties: { title: name } };
    markers.push(marker);
    markers = markers;
  }

  const removePoint = marker => {
    map.removeLayer(marker);
    markers.splice(markers.indexOf(marker), 1);
    markers = markers;
  }

  const download = () => {
    const json = markers.filter(m => m !== currentMarker).map(m => m.toGeoJSON());
    const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(json));
    const a = document.createElement('a');
    a.setAttribute('href', dataStr);
    a.setAttribute('download', 'points.json');
    a.click();
  }

  onMount(currentLocation.init);
</script>

<svelte:head>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.3/dist/leaflet.css" integrity="sha256-kLaT2GOSpHechhsozzB+flnD+zUyjE2LlfWPgU04xyI=" crossorigin=""/>
  <script src="https://unpkg.com/leaflet@1.9.3/dist/leaflet.js" integrity="sha256-WBkoXOwTeyKclOHuWtc+i2uENFpDZ9YPdf5Hf+D7ewM=" crossorigin=""></script>
</svelte:head>

<div class="p-10">
  <h1 class="text-2xl mb-4">Augmented Tours acquire geolocations app</h1>

  <div class="flex-none lg:flex gap-8">
    <div id="map-container" class="w-full h-[460px]" bind:this={ mapContainer }>
      &nbsp;
    </div>

    <div class="w-full">
      <h2 class="text-xl mb-4">Points of interest</h2>
      <div class="form-control w-full mb-4">
        <div class="input-group">
          <input id="name" type="text" placeholder="Point's name" class="input input-bordered w-full" bind:value={ name } />
          <button class="btn btn-outline" on:click={ addPoint }>Add</button>
        </div>
      </div>
      {#if markers.length > 1}
        <div>
          <table class="table w-full">
            <thead>
              <tr><td>Name</td><td>Lat</td><td>Lon</td><td></td></tr>
            </thead>
            <tbody>
              {#each markers.filter(m => m !== currentMarker) as marker}
                <tr>
                  <td>{ marker.feature.properties.title }</td>
                  <td>{ marker.getLatLng().lat }</td>
                  <td>{ marker.getLatLng().lng }</td>
                  <!-- svelte-ignore missing-declaration -->
                  <!-- svelte-ignore a11y-click-events-have-key-events -->
                  <td align="right" class="cursor-pointer" on:click={ () => { removePoint(marker) } }>
                    <i class="fa-regular fa-trash-can"></i>
                  </td>
                </tr>
              {/each}
            </tbody>
            <tfoot>
              <tr>
                <td colspan="4" align="right">
                  <button class="btn" on:click={ download }>Download</button>
                </td>
              </tr>
            </tfoot>
          </table>
        </div>
      {/if}
    </div>
  </div>

  <div>
    { JSON.stringify($currentLocation) }
  </div>
</div>