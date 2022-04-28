<template>
  <div id="app">
    <img id="loading-indicator"
         src="assets/spinner.svg" class="bass">
    <div class="bass venue"></div>
    <div id="widget-container" class="bass seats"
         style="display: none"></div> <button id="discardArea"
                                              style="display: none">OTRA AREA</button>
  </div>
</template>

<script>
export default {
  name: 'App',
  components: {},
  data() {
    return {
      seats: {}
    }
  },
  mounted() {
    /* eslint-disable */
    BeTIX.domReady.then(() => {
      const spinner$ = BeTIX.jQuery('#loading-indicator');
      const venue$ = BeTIX.jQuery('.venue');
      const seats$ = BeTIX.jQuery('.seats');
      const discardArea$ = BeTIX.jQuery('#discardArea');
      const baseUrl = 'assets/';
      const bass = BeTIX.AdvancedSeatSelection;
      const self = this;
      bass.config({
        seats: {
          container: 'widget-container', callbacks: {
            select: unselectPrevious,
          },
          assets: {
            baseUrl,
          }
        },
        venue: {
          venueId: '00-00085H9',
          target: '.bass.venue',
          assets: {
            baseUrl,
          },
          availableAreas,
        },
      });
      bass.seats.channel.on('started', () => spinner$.show());
      bass.seats.channel.on('ready', () => {
        spinner$.hide();
        setupDiscardButton();
      });
      bass.seats.on('create-stage', availableSeats);
      bass.venue.channel.on('started', () => spinner$.show());
      bass.venue.channel.on('ready', () => spinner$.hide());
      bass.venue.channel.on('userSelectedArea', (areaData) => {
        showSeats();
        bass.seats.init({
          venue: '00-00085H9',
          block: areaData.blocks[0],
        });
      });
      bass.venue.init();
      console.log(bass);

// ----------------------------------------------------------------------
      function availableAreas() {
        return BeTIX.jQuery.getJSON(`${baseUrl}/00-00085H9/venue.json`)
            .then(data => data.areas.map(area => area.id));
      }

      function availableSeats() {
        var activeSeatsRatio = .75;
        const allSeats = bass.seats.getModel();
        const someSeats = allSeats.filter(() =>
            Math.random() < activeSeatsRatio);
        bass.seats.seatsAvailable(someSeats);
        console.log(someSeats, allSeats);
        self.seats = someSeats;
      }

      function unselectPrevious(lastSelectedId) {
        var selectedIds = bass.seats.getSelectedSeats();
        selectedIds.forEach(selectedId => {
          if (selectedId !== lastSelectedId) {
            bass.seats.unSelect(selectedId);
          }
        });
      }

      function setupDiscardButton() {
        discardArea$.appendTo('.seats').on('click', () => {
          seats$.empty();
          showVenue();
        })
      }

      function showVenue() {
        venue$.show();
        seats$.hide();
        discardArea$.hide();
      }

      function showSeats() {
        venue$.hide();
        seats$.show();
        discardArea$.show();
      }
    });
  }
}
</script>

<style>
body,
html {
  height: 100%;
  margin: 0;
  padding: 0;
}

#app {
  width: 100%;
  height: 100%;
}

.wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  position: relative;
  background-image: url(/assets/spinner.svg);
  background-position: center center;
  background-repeat: no-repeat;
}

.bass {
  width: 100%;
  height: 80%;
  position: relative;
}

.venue .selectable:not(.disabled) {
  stroke: navy;
  stroke-width: 2px;
  transition: 0.2s all;
  cursor: pointer;
}

.venue .selectable:not(.disabled):hover {
  stroke-width: 4px;
  fill: hsla(0, 0%, 100%, .25);
}

.venue .selectable:not(.disabled).active {
  fill: rgba(0, 123, 255, .5);
}

.venue .disabled {
  fill: rgba(0, 0, 0, .75);
  cursor: not-allowed;
}

.venue svg {
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  position: absolute;
}
</style>
