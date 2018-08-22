FROM dpokidov/gopnik

RUN apt-get update && \
    apt-get install -y git curl unzip node-carto mapnik-utils fontconfig && \
    rm -rf /var/lib/apt/lists/*

COPY fonts/VLGothic/VL-Gothic-Regular.ttf /usr/share/fonts/truetype
COPY fonts/VLGothic/VL-PGothic-Regular.ttf /usr/share/fonts/truetype

WORKDIR /
RUN wget https://github.com/adobe-fonts/source-sans-pro/archive/2.020R-ro/1.075R-it.tar.gz && \
    tar -xzvf 1.075R-it.tar.gz && \
    mkdir /usr/share/fonts/truetype/sourcepro-ttf/ && \
    cp source-sans-pro-2.020R-ro-1.075R-it/TTF/*.ttf /usr/share/fonts/truetype/sourcepro-ttf/ && \
    fc-cache && \
    rm 1.075R-it.tar.gz && \
    rm -rf source-sans-pro-2.020R-ro-1.075R-it

RUN git clone https://github.com/akikinyan/openstreetmap-carto.git
WORKDIR /openstreetmap-carto

# shape data download
ENV UNZIP_OPTS -qqun
ENV SHAPES_DIR data

# create and populate data dir
RUN mkdir -p ${SHAPES_DIR} && \
    mkdir -p ${SHAPES_DIR}/world_boundaries && \
    mkdir -p ${SHAPES_DIR}/simplified-water-polygons-complete-3857 && \
    mkdir -p ${SHAPES_DIR}/ne_110m_admin_0_boundary_lines_land && \
    mkdir -p ${SHAPES_DIR}/water-polygons-split-3857

# world_boundaries
RUN curl -z "${SHAPES_DIR}/world_boundaries-spherical.tgz" -L -o "${SHAPES_DIR}/world_boundaries-spherical.tgz" "http://planet.openstreetmap.org/historical-shapefiles/world_boundaries-spherical.tgz"
RUN tar -xzf ${SHAPES_DIR}/world_boundaries-spherical.tgz -C ${SHAPES_DIR}/

# simplified-water-polygons-complete-3857
RUN curl -z "${SHAPES_DIR}/simplified-water-polygons-complete-3857.zip" -L -o "${SHAPES_DIR}/simplified-water-polygons-complete-3857.zip" "http://data.openstreetmapdata.com/simplified-water-polygons-complete-3857.zip"
RUN unzip $UNZIP_OPTS ${SHAPES_DIR}/simplified-water-polygons-complete-3857.zip \
  simplified-water-polygons-complete-3857/simplified_water_polygons.shp \
  simplified-water-polygons-complete-3857/simplified_water_polygons.shx \
  simplified-water-polygons-complete-3857/simplified_water_polygons.prj \
  simplified-water-polygons-complete-3857/simplified_water_polygons.dbf \
  simplified-water-polygons-complete-3857/simplified_water_polygons.cpg \
  -d ${SHAPES_DIR}/

# ne_110m_admin_0_boundary_lines_land
RUN curl -z ${SHAPES_DIR}/ne_110m_admin_0_boundary_lines_land.zip -L -o ${SHAPES_DIR}/ne_110m_admin_0_boundary_lines_land.zip http://www.naturalearthdata.com/http//www.naturalearthdata.com/download/110m/cultural/ne_110m_admin_0_boundary_lines_land.zip
RUN unzip $UNZIP_OPTS ${SHAPES_DIR}/ne_110m_admin_0_boundary_lines_land.zip \
  ne_110m_admin_0_boundary_lines_land.shp \
  ne_110m_admin_0_boundary_lines_land.shx \
  ne_110m_admin_0_boundary_lines_land.prj \
  ne_110m_admin_0_boundary_lines_land.dbf \
  -d ${SHAPES_DIR}/ne_110m_admin_0_boundary_lines_land/

# water-polygons-split-3857
RUN curl -z "${SHAPES_DIR}/water-polygons-split-3857.zip" -L -o "${SHAPES_DIR}/water-polygons-split-3857.zip" "http://data.openstreetmapdata.com/water-polygons-split-3857.zip"
RUN unzip $UNZIP_OPTS ${SHAPES_DIR}/water-polygons-split-3857.zip \
  water-polygons-split-3857/water_polygons.shp \
  water-polygons-split-3857/water_polygons.shx \
  water-polygons-split-3857/water_polygons.prj \
  water-polygons-split-3857/water_polygons.dbf \
  water-polygons-split-3857/water_polygons.cpg \
  -d ${SHAPES_DIR}/

# antarctica-icesheet-polygons-3857
RUN curl -z "${SHAPES_DIR}/antarctica-icesheet-polygons-3857.zip" -L -o "${SHAPES_DIR}/antarctica-icesheet-polygons-3857.zip" "http://data.openstreetmapdata.com/antarctica-icesheet-polygons-3857.zip"
RUN unzip $UNZIP_OPTS ${SHAPES_DIR}/antarctica-icesheet-polygons-3857.zip \
  antarctica-icesheet-polygons-3857/icesheet_polygons.shp \
  antarctica-icesheet-polygons-3857/icesheet_polygons.shx \
  antarctica-icesheet-polygons-3857/icesheet_polygons.prj \
  antarctica-icesheet-polygons-3857/icesheet_polygons.dbf \
  -d ${SHAPES_DIR}/

# antarctica-icesheet-outlines-3857
RUN curl -z "${SHAPES_DIR}/antarctica-icesheet-outlines-3857.zip" -L -o "${SHAPES_DIR}/antarctica-icesheet-outlines-3857.zip" "http://data.openstreetmapdata.com/antarctica-icesheet-outlines-3857.zip"
RUN unzip $UNZIP_OPTS ${SHAPES_DIR}/antarctica-icesheet-outlines-3857.zip \
  antarctica-icesheet-outlines-3857/icesheet_outlines.shp \
  antarctica-icesheet-outlines-3857/icesheet_outlines.shx \
  antarctica-icesheet-outlines-3857/icesheet_outlines.prj \
  antarctica-icesheet-outlines-3857/icesheet_outlines.dbf \
  -d ${SHAPES_DIR}/

# delete download files
RUN rm ${SHAPES_DIR}/*.zip && rm ${SHAPES_DIR}/*.tgz

# shapeindex
RUN shapeindex --shape_files \
    data/simplified-water-polygons-complete-3857/simplified_water_polygons.shp \
    data/water-polygons-split-3857/water_polygons.shp \
    data/antarctica-icesheet-polygons-3857/icesheet_polygons.shp \
    data/antarctica-icesheet-outlines-3857/icesheet_outlines.shp \
    data/ne_110m_admin_0_boundary_lines_land/ne_110m_admin_0_boundary_lines_land.shp

RUN mkdir /map_data
VOLUME /map_data

RUN mkdir /scripts
COPY scripts/compile_style.sh /scripts
COPY scripts/run_render.sh /scripts
COPY entrypoint.sh /

RUN chmod 755 /scripts/*.sh && \
    chmod 755 /entrypoint.sh

WORKDIR /
ENTRYPOINT /entrypoint.sh
