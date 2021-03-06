# Vincenty

Let's reproduce the results from this table in Vincenty's paper, finding
solutions to the inverse problem.

```
TABLE II - RESULTS OF SOLUTIONS

Line       Φ₁                   Φ₂
           α₁                   L
           s                    α₂

(a) 55° 45' 00.00000"   -33° 26' 00.00000"
    96  36  08.79960    108  13  00.00000"
      14110526.170m     137  52  22.01454

(b) 37° 19' 54.95367"    26° 07' 42.83946"
    95  27  59.63089     41  28  35.50729
       4085966.703m     118  05  58.96161

(c) 35° 16' 11.24862"    67° 22' 14.77638"
    15  44  23.74850    137  47  28.31435
       8084823.839m     144  55  39.92147

(d)  1° 00' 00.00000"    -0° 59' 53.83076"
    89  00  00.00000    179  17  48.02997
      19960000.000m      91  00  06.11733

(e)  1° 00' 00.00000"     1° 01' 15.18952"
     4  59  59.99995    179  46  17.84244
      19780006.558m     174  59  59.88481

Notation
    Φ₁,Φ₂ - geodetic latitudes at points P₁ and P₂
    α₁,α₂ - azimuths of the geodesic; α₂ in the direction P₁ and P₂
       s  - length of the geodesic

```

```ucm
.> cd .flight.Geodesy

```
```unison
---
title: ellipsoids.u
---
> bessel
> hayford

```


```ucm

  ✅
  
  ellipsoids.u changed.
  
  Now evaluating any watch expressions (lines starting with
  `>`)... Ctrl+C cancels.

    1 | > bessel
          ⧩
          Ellipsoid (Radius 6377397.155) 299.1528128
  
    2 | > hayford
          ⧩
          Ellipsoid (Radius 6378388.0) 297.0

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

es : [Ellipsoid]
es = [bessel, hayford, hayford, hayford, hayford]

inversePoints : ((DMS, DMS), (DMS, DMS)) -> InverseProblem LatLng
inversePoints dmsLatLng =
    ((xLat, xLng), (yLat, yLng)) = dmsLatLng

    (Rad xLat') = Convert.degToRad (Deg (toDeg xLat))
    (Rad xLng') = Convert.degToRad (Deg (toDeg xLng))
    
    (Rad yLat') = Convert.degToRad (Deg (toDeg yLat))
    (Rad yLng') = Convert.degToRad (Deg (toDeg yLng))

    x = LatLng (Lat xLat') (Lng xLng')
    y = LatLng (Lat yLat') (Lng yLng')

    InverseProblem x y

solvedDistances : [Optional Float]
solvedDistances =
    f e dmsLatLng =
      (InverseProblem x y) = inversePoints dmsLatLng
      Vincenty.distance e x y
      
    List.zipWith f es xys

solvedAzimuthFwd : [Optional Rad]
solvedAzimuthFwd =
    use Lat Lat
    use Lng Lng
    f e dmsLatLng =
      (InverseProblem x y) = inversePoints dmsLatLng
      Vincenty.azimuthFwd e x y
    List.zipWith f es xys

solvedAzimuthRev : [Optional Rad]
solvedAzimuthRev =
    use Lat Lat
    use Lng Lng
    f e dmsLatLng =
      (InverseProblem x y) = inversePoints dmsLatLng
      Vincenty.azimuthRev e x y
    List.zipWith f es xys

> somes solvedDistances
> List.map DMS.fromRad (somes solvedAzimuthFwd)
> List.map DMS.fromRad (somes solvedAzimuthRev)

```


```ucm

  I found and typechecked these definitions in vincenty.u. If
  you do an `add` or `update`, here's how your codebase would
  change:
  
    ⍟ These new definitions are ok to `add`:
    
      es               : [Ellipsoid]
      inversePoints    : ((DMS, DMS), (DMS, DMS))
                         -> InverseProblem LatLng
      solvedAzimuthFwd : [Optional Rad]
      solvedAzimuthRev : [Optional Rad]
      solvedDistances  : [Optional Float]
      xys              : [((DMS, DMS), (DMS, DMS))]
  
  Now evaluating any watch expressions (lines starting with
  `>`)... Ctrl+C cancels.

    59 | > somes solvedDistances
           ⧩
           [ 1.411052616959625e7,
             4085966.7026130124,
             8084823.83829748,
             1.9877285829416234e7,
             1.9780006558786154e7 ]
  
    60 | > List.map DMS.fromRad (somes solvedAzimuthFwd)
           ⧩
           [ DMS (+96, +36, 8.79959547673593),
             DMS (+95, +27, 59.63088839623538),
             DMS (+15, +44, 23.74849756849912),
             DMS (+88, +59, 59.99897064494121),
             DMS (+4, +59, 59.999954187251205) ]
  
    61 | > List.map DMS.fromRad (somes solvedAzimuthRev)
           ⧩
           [ DMS (+137, +52, 22.01453494489442),
             DMS (+118, +5, 58.96160847082683),
             DMS (+144, +55, 39.921473003585106),
             DMS (+91, +0, 6.118356269087144),
             DMS (+174, +59, 59.88480239330784) ]

```
