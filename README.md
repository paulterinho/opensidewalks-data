# opensidewalks-data

(Forked) Extending the opensidewalks data project to also include instructions for exporting it to a Postgis Database. See Appendix for more information.

Build workflows for OpenSidewalks data releases. Workflows generate OpenSidewalks
Schema reference implementations for multiple cities.

## Installation

### Docker-based installation

1. Build the container
    `docker build -t opensidewalks-data` .

## Extract a single region's data

Running these commands will create 
- the PBF files (intermediary files) into the `./cities/{cityName}/data_sources/` folder
- the `transportation.geojson` and `regions.geojson` in the `./cities/{cityName}/output/` folder

Commands:

    docker run --rm -v $(pwd):/data opensidewalks-data bash -c "cd /data/cities/seattle && snakemake -j 8 --snakefile ./Snakefile.fetch"
    docker run --rm -v $(pwd):/data opensidewalks-data bash -c "cd /data/cities/seattle && snakemake -j 8 --snakefile ./Snakefile"

## Extract all regions and transform into `transportation.geojson` and `regions.geojson`

Check out the `merge.py` file. This will merge the `transportation.geojson` of the different cities into one file.

    docker run --rm -v $(pwd):/data opensidewalks-data bash -c "cd /data && python ./merge.py"

# OpenSidewalks Data Schema

See the [schema repo](https://github.com/OpenSidewalks/OpenSidewalks-Schema).

# Appendix
## Notes for adding a new city
Notes for the progress on this forked repository.

One thing to note is that with Calgary, the DEM wasn't available online synchronously, so I committed it to this repo and am fetching it that way from the VM. Not ideal, but it works.

Next, follow the two VM invocations to create the `.geojson` files we need to import into a database. This will output to the `cities/calgary/output` folder.

    docker run --rm -v $(pwd):/data opensidewalks-data bash -c "cd /data/cities/calgary && snakemake -j 8 --snakefile ./Snakefile.fetch"
    docker run --rm -v $(pwd):/data opensidewalks-data bash -c "cd /data/cities/calgary && snakemake -j 8 --snakefile ./Snakefile"

## 1) Create a Postgis DB
The resulting GeoJson and PBF files are too big to inspect with most viewers. We can push it into a database and inspect it that way.

Also, tools like QGis can connect to a DB for inspecting the data.

#### Steps
- Install Postgres
- Create a database via `createdb dbname`
- use psql to log in to the new db and install Postgis via `CREATE EXTENSION postgis;`

## 2) (Optional) Load the PBF file to see what it contains
It's possible to load the PBF zip file to see what the first invocation of this VM is doing (aquiring the OSM data)

#### Steps
- Install `osm2pgsql`. This allows us to push the contents of a PBF file (zip) into a Postgis database 
- `osm2pgsql -c -d dbname -U username -H localhost -S default.styles.style extract.osm.pbf`

## 3) Load the .geojson into a Postgis DB

Go to the `output` directory in `calgary`

#### Steps
- Install `ogr2ogr`. It allows us to push a `.geojson` file into a Postgis database one table at a time. 
- `ogr2ogr -f "PostgreSQL" PG:"dbname=dbname user=username" "barriers.geojson" -nln barriers -append`
- `ogr2ogr -f "PostgreSQL" PG:"dbname=dbname user=username" "barriers.geojson" -nln barriers.geojson -append`

## 4) Use DBeaver or PgAdmin4's spatial viewer to see if you chose your bounding box correctly.
This allows us a granular way of verifying our geometries are correct.

Check out `https://dbeaver.io/` for the download.

## 5) Connect via QGis
You can use QGis to load the database tables as Map Layers.

This also allows you to overlay the DEM you used to verify that it's extent is larger than the bounding box you chose (important because it uses that information to calculate segement inclincation).
