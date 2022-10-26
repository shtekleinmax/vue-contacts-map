<script>
const ymaps = window.ymaps;

export default {
  name: "ContactsMap",
  props: {
    options: {
      type: Object,
      default: function () {
        return {};
      },
    },
  },
  data() {
    return {
      map: this.options,
      id: "contacts-map",
    };
  },
  methods: {
    createYandexMap: function () {
      let vm = this;

      if (vm.map.coord) {
        let point = vm.map.coord.split(",");
        vm.map.coord = [point[0], point[1]];
      }

      if (vm.map.center) {
        let point = vm.map.center.split(",");
        vm.map.center = [point[0], point[1]];
      } else if (vm.map.coord) {
        vm.map.center = vm.map.coord;
      }

      vm.map = Object.assign(
        {
          center: [],
          coord: [],
          zoom: vm.map.zoom,
          placemarks: null,
          iconContent: "",
          balloonContentHeader: "",
          description: "",
          balloonContentFooter: "",
          placemarkStyle: { preset: "twirl#blueStretchyIcon" },
          controls: [
            "typeSelector",
            "zoomControl",
            "smallZoomControl",
            "mapTools",
          ],
          type: "yandex#map",
        },
        vm.map
      );

      // Создаем карту
      var myMap = new ymaps.Map(vm.id, {
        center: vm.map.center,
        zoom: vm.map.zoom,
        type: "yandex#map",
      });

      if (vm.map.iconImage && vm.map.iconWidth && vm.map.iconHeight) {
        vm.map.placemarkStyle = {
          iconLayout: "default#image",
          iconImageHref: vm.map.iconImage,
          iconImageSize: [vm.map.iconWidth, vm.map.iconHeight],
          iconImageOffset: [0, -vm.map.iconHeight],
        };
      }

      if (vm.map.placemarks) {
        let options = vm.map;
        let myCollection = new ymaps.GeoObjectCollection({}, {});
        let placemarks = JSON.parse(vm.map.placemarks);

        placemarks.forEach(function (placemark) {
          let point = placemark.coord.split(",");
          let coord = [point[0], point[1]];
          let title = placemark.title ? placemark.title : "";
          let zoom = placemark.zoom ? placemark.zoom : options.zoom;
          let balloon = {};

          if (placemark.description) {
            balloon = {
              iconContent: "",
              balloonContentHeader: title,
              balloonContentBody:
                '<div class="map-baloon">' + placemark.description + "</div>",
              balloonContentFooter: "",
              hintContent: title,
            };
          } else {
            balloon = { hintContent: title };
          }

          let myPlacemark = new ymaps.Placemark(
            coord,
            balloon,
            options.placemarkStyle
          );
          // $(myPlacemark).data('zoom', this.zoom);

          myPlacemark.events.add(
            "click",
            function () {
              if (zoom > 0) {
                myMap.setCenter(this.geometry.getBounds()[0], zoom, {});
              }
            },
            myPlacemark
          );

          myCollection.add(myPlacemark);
        });

        // Добавляем на карту коллекцию маркеров
        myMap.geoObjects.add(myCollection);

        // Закрытие балуна - центрируем карту по маркерам
        myMap.events.add("balloonclose", function () {
          myMap
            .setBounds(myCollection.getBounds(), {
              checkZoomRange: true,
            })
            .then(function () {
              myMap.setZoom(vm.map.zoom);
            });
        });

        // Центрируем карту по маркерам
        myMap.setBounds(myCollection.getBounds(), {
          checkZoomRange: true,
        });
        /*
        .then(function() {
          myMap.setZoom(vm.map.zoom);
        });
        */
      } else {
        // Создаем одиночный маркер
        var myPlacemark = new ymaps.Placemark(
          vm.map.coord,
          {
            iconContent: vm.map.iconContent,
            balloonContentHeader: vm.map.balloonContentHeader,
            balloonContentBody:
              '<div class="map-baloon">' + vm.map.description + "</div>",
            balloonContentFooter: vm.map.balloonContentFooter,
          },
          vm.map.placemarkStyle
        );

        myPlacemark.events.add(
          "click",
          function () {
            if (vm.map.zoom > 0) {
              myMap.setCenter(this.geometry.getBounds()[0], 17, {});
            }
          },
          myPlacemark
        );

        // Закрытие балуна - центрируем карту по маркерам
        myMap.events.add("balloonclose", function () {
          if (vm.map.center) {
            myMap.setCenter(vm.map.center, vm.map.zoom, {});
          }
        });

        myMap.geoObjects.add(myPlacemark);
      }

      /*
      // Добавляем область заливки
      if (!empty(options.center) && !empty(this.options.fill)) {
        $.each(this.options.fill, function(i) {

          if (!empty(this.border)) {
            var fillColor = this.bgcolor;
            var strokeColor = this.brcolor;
            var hint = this.title;

            var myPolygon = new ymaps.Polygon([
              this.border
            ], {
              hintContent: hint
            }, {
              fillColor: fillColor,
              interactivityModel: 'default#transparent',
              strokeColor: strokeColor,
              strokeWidth: 6,
              opacity: 0.5
            });

            myMap.geoObjects.add(myPolygon);
          }

        });
      }
      */

      return myMap;
    },
  },
  mounted() {
    let vm = this;

    ymaps.ready(function () {
      vm.createYandexMap();
    });
  },
};
</script>

<template>
  <div id="contacts-map" class="contacts-map"></div>
</template>

<style lang="scss">
/* MAPS */
.contacts-map {
  max-width: 560px;
  width: 100%;
  height: $map_height;
  margin: $map_margin;
  display: block;
  flex-shrink: 0;
  background: #f6f6f6;
  position: relative;
  font-size: 0;
  border-radius: 30px;
  overflow: hidden;

  &::before {
    content: ".";
    display: block;
    position: absolute;
    left: 50%;
    top: 50%;
    width: 55px;
    height: 55px;
    margin: -30px 0 0 -30px;
    border-radius: 50%;
    border: 5px solid $a_color_hover;
    border-right: 5px solid transparent;
    border-bottom: 5px solid transparent;
    font-size: 0;
    color: transparent;
    -webkit-animation: overlay-spin 3s infinite linear;
    animation: overlay-spin 3s infinite linear;
  }
}

.map-baloon {
  font-size: 14px;
  line-height: 21px;

  p {
    margin: 0;
    padding: 0;
  }
}

.dg-popup__container {
  min-width: 200px;
}

.leaflet-popup-content {
  p {
    margin: 0;
  }
}
</style>
