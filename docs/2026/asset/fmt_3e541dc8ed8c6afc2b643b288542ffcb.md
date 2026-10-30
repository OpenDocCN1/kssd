
```text
.data:0005C0B1                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C0C2                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C0D3                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C0E4                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C0F5                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C106                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C117                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C128                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C139                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C14A                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C15B                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C05C ; char s[65536]
.data:0005C05C s               DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C05C                                         ; DATA XREF: sub_FDE4+4E↑o
.data:0005C05C                                         ; sub_FDE4:loc_FEBA↑o ...
.data:0005C06D                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C07E                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C08F                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C0A0                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C0B1                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C0C2                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C0D3                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C0E4                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C0F5                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C106                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C117                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C128                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C139                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C14A                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
.data:0005C15B                 DCB 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
```

```text
getconf NAME_MAX /
255
getconf PATH_MAX /
4096
getconf NAME_MAX /
255
getconf PATH_MAX /
4096
```

```c
printf_(s, "Java_com_sina_filter_JpegExifExtrator_nativeExifExtrator: parse start");
v15 = TinyEXIF::EXIFInfo::parseFrom((TinyEXIF::EXIFInfo *)&v61, (const unsigned __int8 *)p, v115[0]);
if ( v15 )
{
    printf_(s, "EXIF parse failed err code: %d", v15);
    printf_(s, "ProjectionType %d", HIWORD(v87));
}
else
{
    printf_(s, "EXIF parse success");
    if ( v61 )
        printf_(s, "ImageResolution %d x %d", (_DWORD)v61, HIDWORD(v61));
    if ( v62 )
        printf_(s, "RelatedImageResolution %d x %d ", (_DWORD)v62, HIDWORD(v62));
    if ( *((_DWORD *)v63 - 3) )
        printf_(s, "Description %s", v63);
    if ( *((_DWORD *)v64 - 3) || *((_DWORD *)v65 - 3) )
        printf_(s, "CameraModel make: %s, model: %s", v64, v65);
    printf_(s, "Orientation %d", v66);
    printf_(s, "Resolution %lf x %lf", v67, v68);
    printf_(s, "ResolutionUnit %d", (unsigned __int16)v69);
    printf_(s, "BitsPerSample %d", HIWORD(v69));
    if ( *((_DWORD *)v70 - 3) )
        printf_(s, "Software %s", v70);
    if ( *((_DWORD *)v71 - 3) )
        printf_(s, "DateTime %s", v71);
    if ( *((_DWORD *)v72 - 3) )
        printf_(s, "DateTimeOriginal %s", v72);
    if ( *((_DWORD *)v73 - 3) )
        printf_(s, "DateTimeDigitized %s", v73);
    if ( *((_DWORD *)v74 - 3) )
        printf_(s, "SubSecTimeOriginal %s", v74);
    if ( *((_DWORD *)v75 - 3) )
        printf_(s, "Copyright %s", v75);
    printf_(s, "ExposureTime %0.10lf s", v76);
    printf_(s, "FNumber %lf", v77);
    printf_(s, "ExposureProgram %d", v78);
    printf_(s, "ISOSpeed %d", v79);
    printf_(s, "ShutterSpeedValue %0.10lf", v80);
    printf_(s, "ApertureValue %0.10lf", v81);
    printf_(s, "BrightnessValue %0.10lf", v82);
    printf_(s, "ExposureBiasValue %lf", v83);
    printf_(s, "SubjectDistance %lf", v84);
    printf_(s, "FocalLength %lf mm", v85);
    printf_(s, "Flash %d", (unsigned __int16)v86);
    if ( i_3 != i_1 )
    {
        printf_(s, "SubjectArea: ");
        i_2 = i_3;
        for ( i = i_1; i != i_2; ++i_2 )
            printf_(s, "%d", *i_2);
    }
    printf_(s, "MeteringMode %d", HIWORD(v86));
    printf_(s, "LightSource %d", (unsigned __int16)v87);
    printf_(s, "ProjectionType %d", HIWORD(v87));
    printf_(s, "LensInfo.FStopMin %lf", v91);
    printf_(s, "LensInfo.FStopMax %lf", v92);
    printf_(s, "LensInfo.FocalLengthMin %lf mm", v93);
    printf_(s, "LensInfo.FocalLengthMax %lf mm", v94);
    printf_(s, "LensInfo.DigitalZoomRatio %lf ", v95);
    printf_(s, "LensInfo.FocalLengthIn35mm %lf ", v96);
    printf_(s, "LensInfo.FocalPlaneXResolution %0.10lf", v97);
    printf_(s, "LensInfo.FocalPlaneYResolution %0.10lf ", v98);
    printf_(s, "LensInfo.FocalPlaneResolutionUnit %d", v99);
    if ( *((_DWORD *)v100 - 3) || *((_DWORD *)v101 - 3) )
        printf_(s, "LensInfo.Model %s - %s", v100, v101);
    if ( TinyEXIF::EXIFInfo::Geolocation_t::hasLatLon((TinyEXIF::EXIFInfo::Geolocation_t *)&v102) == 1 )
    {
        printf_(s, "GeoLocation.Latitude %0.10lf", v102);
        printf_(s, "GeoLocation.Longitude %0.10lf", v103);
    }
}
```
