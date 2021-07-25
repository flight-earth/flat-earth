# Vincenty

From the table in Vincenty's paper showing the results of solutions.
 
```ucm
.> cd .flight.Geodesy

```
```unison
---
title: vincenty.u
---
xys : [((DMS, DMS), (DMS, DMS))]
xys =
    [ ((( +55, +45,  0.00000), (  +0,  +0,  0.0)), (( -33, +26,  0.00000), (+108, +13,  0.00000)))
    , ((( +37, +19, 54.95367), (  +0,  +0,  0.0)), (( +26,  +7, 42.83946), ( +41, +28, 35.50729)))
    , ((( +35, +16, 11.24862), (  +0,  +0,  0.0)), (( +67, +22, 14.77638), (+137, +47, 28.31435)))
    , (((  +1,  +0,  0.00000), (  +0,  +0,  0.0)), ((  +0, -59, 53.83076), (+179, +17, 48.02997)))
    , (((  +1,  +0,  0.00000), (  +0,  +0,  0.0)), ((  +1,  +1, 15.18952), (+179, +46, 17.84244)))
    ]
        |> List.map (cases ((xLat, xLng), (yLat, yLng)) ->
            x = (DMS xLat, DMS xLng)
            y = (DMS yLat, DMS yLng)

            (x, y))

es = [bessel, hayford, hayford, hayford, hayford]

ds =
    [ 14110526.170
    ,  4085966.703
    ,  8084823.839
    , 19960000.000
    , 19780006.5584
    ]

fwdAzimuths : [DMS]
fwdAzimuths =
    List.map DMS
        [ ( +96, +36,  8.79960)
        , ( +95, +27, 59.63089)
        , ( +15, +44, 23.74850)
        , ( +89,  +0,  0.00000)
        , (  +4, +59, 59.99995)
        ]

revAzimuths : [DMS]
revAzimuths =
    List.map DMS
        [ (+137, +52, 22.01454)
        , (+118,  +5, 58.96161)
        , (+144, +55, 39.92147)
        , ( +91,  +0,  6.11733)
        , (+174, +59, 59.88481)
        ]

indirectSolutionDistances =
    use Lat Lat
    use Lng Lng
    f e ll =
        ((xLat, xLng), (yLat, yLng)) = ll
        (Rad xLat') = Convert.degToRad (Deg (toDeg xLat))
        (Rad xLng') = Convert.degToRad (Deg (toDeg xLng))
        
        (Rad yLat') = Convert.degToRad (Deg (toDeg yLat))
        (Rad yLng') = Convert.degToRad (Deg (toDeg yLng))
        Vincenty.distance e (LatLng (Lat xLat') (Lng xLng')) (LatLng (Lat yLat') (Lng yLng'))
    List.zipWith f es xys

> somes indirectSolutionDistances

```


```ucm

  I found and typechecked these definitions in vincenty.u. If
  you do an `add` or `update`, here's how your codebase would
  change:
  
    ⍟ These new definitions are ok to `add`:
    
      ds                        : [Float]
      es                        : [Ellipsoid]
      fwdAzimuths               : [DMS]
      indirectSolutionDistances : [Optional Float]
      revAzimuths               : [DMS]
      xys                       : [((DMS, DMS), (DMS, DMS))]
  
  Now evaluating any watch expressions (lines starting with
  `>`)... Ctrl+C cancels.

    58 | > somes indirectSolutionDistances
           ⧩
           [ 1.411052616959625e7,
             4085966.7026130124,
             8084823.83829748,
             1.9877285829416234e7,
             1.9780006558786154e7 ]

```