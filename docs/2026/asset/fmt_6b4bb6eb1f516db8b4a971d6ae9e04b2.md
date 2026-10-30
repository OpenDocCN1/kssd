
```c
if ( TinyEXIF::EXIFInfo::Geolocation_t::hasAltitude((TinyEXIF::EXIFInfo::Geolocation_t *) &v102) == 1 )
{
printf_(s, "GeoLocation.Altitude %lf m", v104);
printf_(s, "GeoLocation.AltitudeRef %d", v105);
}
if ( TinyEXIF::EXIFInfo::Geolocation_t::hasRelativeAltitude((TinyEXIF::EXIFInfo::Geolocation_t *) &v102) == 1 )
printf_(s, "GeoLocation.RelativeAltitude %lf", v106);
if ( TinyEXIF::EXIFInfo::Geolocation_t::hasOrientation((TinyEXIF::EXIFInfo::Geolocation_t *) &v102) == 1 )
{
printf_(s, "GeoLocation.RollDegree %lf", v107);
printf_(s, "GeoLocation.PitchDegree %lf", v108);
printf_(s, "GeoLocation.YawDegree %lf", v109);
}
printf_(s, "GeoLocation.GPSDOP %lf", v110);
printf_(s, "GeoLocation.GPSDifferential %d", v111);
if ( *((_DWORD *)v112 - 3) )
printf_(s, "GeoLocation.GPSMapDatum %s", v112);
if ( *((_DWORD *)v113 - 3) )
printf_(s, "GeoLocation.GPSTimeStamp %s", v113);
if ( *((_DWORD *)v114 - 3) )
printf_(s, "GeoLocation.GPSDateStamp %s", v114);
}
if ( p )
operator delete(p);
printf_(s, "Java_com_sina_filter_JpegExifExtractor_nativeExifExtrator exif obj setup start");
printf_(s, "Java_com_sina_filter_JpegExifExtractor_nativeExifExtrator: parse start");
v15 = TinyEXIF::EXIFInfo::parseFrom((TinyEXIF::EXIFInfo *) &v61, (const unsigned __int8 *)p, v115[0]);
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
if ( TinyEXIF::EXIFInfo::Geolocation_t::hasLatLon((TinyEXIF::EXIFInfo::Geolocation_t *) &v102) == 1 )
{
printf_(s, "GeoLocation.Latitude %0.10lf", v102);
printf_(s, "GeoLocation.Longitude %0.10lf", v103);
}
if ( TinyEXIF::EXIFInfo::Geolocation_t::hasAltitude((TinyEXIF::EXIFInfo::Geolocation_t *) &v102) == 1 )
{
printf_(s, "GeoLocation.Altitude %lf m", v104);
printf_(s, "GeoLocation.AltitudeRef %d", v105);
}
if ( TinyEXIF::EXIFInfo::Geolocation_t::hasRelativeAltitude((TinyEXIF::EXIFInfo::Geolocation_t *) &v102) == 1 )
printf_(s, "GeoLocation.RelativeAltitude %lf", v106);
if ( TinyEXIF::EXIFInfo::Geolocation_t::hasOrientation((TinyEXIF::EXIFInfo::Geolocation_t *) &v102) == 1 )
{
printf_(s, "GeoLocation.RollDegree %lf", v107);
printf_(s, "GeoLocation.PitchDegree %lf", v108);
printf_(s, "GeoLocation.YawDegree %lf", v109);
}
printf_(s, "GeoLocation.GPSDOP %lf", v110);
printf_(s, "GeoLocation.GPSDifferential %d", v111);
if ( *((_DWORD *)v112 - 3) )
printf_(s, "GeoLocation.GPSMapDatum %s", v112);
if ( *((_DWORD *)v113 - 3) )
printf_(s, "GeoLocation.GPSTimeStamp %s", v113);
if ( *((_DWORD *)v114 - 3) )
printf_(s, "GeoLocation.GPSDateStamp %s", v114);
}
if ( p )
operator delete(p);
printf_(s, "Java_com_sina_filter_JpegExifExtractor_nativeExifExtrator exif obj setup start");
printf_(s, "CameraModel make: %s, model: %s", v64, v65);
printf_(s, "CameraModel make: %s, model: %s", v64, v65);
```

Arch: arm-32-little
RELRO: Full RELRO
Stack: Canary found
NX: NX enabled
PIE: PIE enabled
Arch: arm-32-little

[回复或点赞可查看完整内容](https://bbs.kanxue.com/#quick_reply_form)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2025-10-11 16:11

被易之生生编辑，原因：

[#缓冲区溢出](https://bbs.kanxue.com/forum-150-1-156.htm)
[#Linux](https://bbs.kanxue.com/forum-150-1-161.htm)
[#Andorid](https://bbs.kanxue.com/forum-150-1-162.htm)
