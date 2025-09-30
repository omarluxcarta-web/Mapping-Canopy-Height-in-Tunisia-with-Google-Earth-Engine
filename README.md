
var canopy_vis = {
  min: 0,
  max: 65,
  palette: [
    "#010005","#150b37","#3b0964","#61136e","#85216b",
    "#a92e5e","#cc4248","#e75e2e","#f78410","#fcae12",
    "#f5db4c","#fcffa4"
  ]
};

var sd_vis = {
  min: 0,
  max: 15,
  palette: [
    "#0d0406","#241628","#36274d","#403a76","#3d5296",
    "#366da0","#3488a6","#36a2ab","#44bcad","#6dd3ad",
    "#aee3c0","#def5e5"
  ]
};

var canopy_height = ee.Image("users/nlang/ETH_GlobalCanopyHeight_2020_10m_v1");
var standard_deviation = ee.Image("users/nlang/ETH_GlobalCanopyHeightSD_2020_10m_v1");

var isoDict = {
  "TUN": "Tunisia"
};
var countries = ee.FeatureCollection("FAO/GAUL_SIMPLIFIED_500m/2015/level0");

function showCountry(isoCode) {
  var countryName = isoDict[isoCode];
  var country = countries.filter(ee.Filter.eq('ADM0_NAME', countryName));

  var canopy_clipped = canopy_height.clip(country);
  var sd_clipped = standard_deviation.clip(country);

  Map.centerObject(country, 6);
  Map.setOptions("SATELLITE");
  Map.clear();

  Map.addLayer(canopy_clipped, canopy_vis, "Hauteur de la canopée - " + countryName);
  Map.addLayer(sd_clipped, sd_vis, "Incertitude de mesure - " + countryName);

  addLegend("Hauteur de canopée (m)", canopy_vis.palette, canopy_vis.min, canopy_vis.max, 'bottom-left');
  addLegend("Écart-type (m)", sd_vis.palette, sd_vis.min, sd_vis.max, 'bottom-right');

  Export.image.toDrive({
    image: canopy_clipped,
    description: 'CanopyHeight_' + isoCode,
    folder: 'Clearinghouse_Tunisia',
    fileNamePrefix: 'CanopyHeight_' + isoCode,
    region: country.geometry(),
    scale: 10,
    crs: 'EPSG:4326',
    maxPixels: 1e13
  });

  Export.image.toDrive({
    image: sd_clipped,
    description: 'CanopyHeightSD_' + isoCode,
    folder: 'Clearinghouse_Tunisia',
    fileNamePrefix: 'CanopyHeightSD_' + isoCode,
    region: country.geometry(),
    scale: 10,
    crs: 'EPSG:4326',
    maxPixels: 1e13
  });
}

function addLegend(title, palette, min, max, position) {
  var legend = ui.Panel({style: {position: position, padding: '8px 15px'}});
  legend.add(ui.Label({value: title, style: {fontWeight: 'bold', fontSize: '14px'}}));

  var makeColorBar = function(palette) {
    return ui.Thumbnail({
      image: ee.Image.pixelLonLat().select(0).multiply((max - min) / 100).add(min),
      params: {bbox: [0, 0, 100, 10], dimensions: '100x10', min: min, max: max, palette: palette},
      style: {stretch: 'horizontal', margin: '0px 8px', maxHeight: '24px'}
    });
  };

  var legendLabels = ui.Panel({
    widgets: [
      ui.Label(min.toString()),
      ui.Label((max/2).toString(), {stretch: 'horizontal', textAlign: 'center'}),
      ui.Label(max.toString())
    ],
    layout: ui.Panel.Layout.flow('horizontal')
  });

  legend.add(makeColorBar(palette));
  legend.add(legendLabels);
  Map.add(legend);
}

showCountry("TUN");
