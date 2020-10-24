# dirtymapview


https://github.com/developmentseed/dirty-reprojectors


Just a rough example for now, not sure this is worthwhile or just a obscure plainview: 

```R
library(sf)
library(raster)
#ant <- sfdct::antarctica[1, ]
ice <- sf::st_as_sf(SOmap::SOmap_data$seaice_oct) ## openstack is bork today: raadtools::readice(latest = TRUE)
ant <- sf::st_transform(sfdct::antarctica[1, ], raster::projection(ice))

plot(ant$geometry)

dirty_merc <- function() {
  "+proj=merc +a=6378137.0 +b=6378137.0"
}
## your projection with +merc sensibilities (maxext: anything <= 20037508)
dirty_scl <- function(x, maxext = 20037508, xlim = NULL, ylim = NULL) {
  A <- maxext
  if (is.null(xlim)) {
    xlim <- spex::xlim(x)
  }
  if (is.null(ylim)) {
    ylim <- spex::ylim(x)
  }
  df <- sfheaders::sf_to_df(x)
  df$x <- scales::rescale(df$x, to = c(-A, A), from = xlim)
  df$y <- scales::rescale(df$y, to = c(-A, A), from = ylim)
  out <- switch(as.character(st_geometry_type(x)), 
         MULTIPOLYGON = sfheaders::sf_mpoly(df), 
         MULTILINESTRING = sfheaders::sf_mline(df))
  suppressWarnings(sf::st_set_crs(out, sf::st_crs(x)))
  }
dirty_reproj <- function(x) {
  suppressWarnings(sf::st_transform(sf::st_set_crs(x, sf::st_crs(dirty_merc())), 
                   4326))
}

xlim0 <- spex::xlim(ice)
ylim0 <- spex::ylim(ice)
A <- 20037508/5
dirty_ant <- dirty_reproj(dirty_scl(ant, maxext = A, xlim = xlim0, ylim = ylim0))  ## merc sensibilities in longlat
dirty_ice <- dirty_reproj(dirty_scl(ice, maxext = A, xlim = xlim0, ylim= ylim0))  ## merc sensibilities in longlat
proj_extent <- spex::spex(st_transform(dirty_ice$geometry, dirty_merc()))
dirty_topo <- setExtent(raster::projectRaster(anglr::gebco, 
                    raster::raster(extent(xlim0, ylim0), crs = projection(ice), 
                                   nrows = 256, ncols = 256)), proj_extent)

projection(dirty_topo) <- dirty_merc()
library(mapview)
mapview(dirty_topo) + mapview(dirty_ant) + mapview(dirty_ice)

```

![ A test image](https://github.com/mdsumner/dirtymapview/raw/master/Untitled.png)
