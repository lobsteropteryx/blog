+++
date = "2013-09-30T00:00:00+00:00"
draft = false
title = "Numbering Points within a Polygon"
+++

This is the sort of thing that seems really easy in hindsight, but it took me a little while to get right.

There are lots of things that I don't like about PL/SQL, but I often prefer to do spatial operations at the table level inside the database, rather than using ArcPy or ArcObjects; I think that it's easier to reuse queries like this, the code tends to be more readable and concise, and performance is almost always significantly better.

In this case, we had  a set of polygons ("PPAs") that needed to be assigned a value based on the county they fell in, as well as being uniquely numbered within that county.

<code>
        select /*+ ordered use_nl (t,c) use_nl (t,p) */
        objectid,
        state,
        county_name county,
        region,
        row_number() over (partition by region, state, county_name
        order by region, state, county_name,
        sdo_geom.sdo_centroid(p.shape, 0.005).sdo_point.x,
        sdo_geom.sdo_centroid(p.shape, 0.005).sdo_point.y) as rnum
        from
        (
          select p.objectid, p.traps, c.state, c.county_name, p.region, sdo_geom.sdo_centroid(p.shape, 0.005) as shape
          from table(sdo_join('PPAS','SHAPE','COUNTIES','SHAPE')) t,
            ppas p,
            counties c
          where t.rowid1 = p.rowid
          and t.rowid2 = c.rowid
          and c.state = p.state
          and sdo_geom.relate(c.shape, 'anyinteract', sdo_geom.sdo_centroid(p.shape, 0.005), 0.005) = 'TRUE'
        ) p
        order by state, county_name, rnum
</code>

Ordering by the centroid X and Y values in the partition clause is important; without that, the polygons don't get numbered in a deterministic way.  In this case, the polygons were generated daily by a decision support system, and without ordering the blocks spatially, the numbers within each county would be swapped around each day.
