# android_vendor_jancar_ac8257_demo

Fichiers propriétaires côté `/system` pour **LineageOS 16.0** sur l'autoradio Jancar **UJC201** (AC8257), à placer
dans `vendor/jancar/ac8257_demo` (arbre : [android_device_alps_ac8257_demo](https://github.com/LibreHU/android_device_alps_ac8257_demo),
branche `lineage-16.0`, guide `docs/BUILD.md`). Branche `lineage-16.0`. Makefiles écrits par `setup-makefiles.sh` de
l'arbre (`extract_utils.sh` de LineageOS 16.0).

La partition vendor stock entière (`vendor.img`, 2 Gio) se pose aussi dans ce dossier mais **n'est pas dans le dépôt**
(trop grosse, propre à chaque unité : `.gitignore`).

## Contenu

| Fichier | Source | Rôle |
|---|---|---|
| `proprietary/framework/autochips.jar` | ROM stock `UJC201-V1.151R6` : `system/framework/autochips.jar` (vide) + `boot-autochips.vdex` | jar du BOOTCLASSPATH (`PRODUCT_BOOT_JARS += autochips`) : `AtcMetazone`, `AtcLogo`, `BackCar`, `LocalBluetoothManager`, CarPlay / iAP2, DVR, AVM… (418 classes) |
| `proprietary/lib{,64}/vendor.autochips.hardware.*@*.so` (9 en 32 bits, 10 en 64 bits) | même ROM, `/system/lib{,64}` | interfaces HIDL des HAL Autochips du vendor (côté clients) |
| `proprietary/lib{,64}/lib{atcavm,atcvobsub,dvr,logo,metazone}_jni.so` | même ROM | bibliothèques JNI de `autochips.jar` |
| `proprietary/lib{,64}/libusbdvr.so` | même ROM | dépendance de `libdvr_jni.so` (enregistrement USB) |

Toutes les autres dépendances (`NEEDED`) de ces fichiers sont dans LineageOS 16.0 (`libhidltransport`,
`libmedia_omx`, `libstagefright_foundation`, `libstdc++`, `android.hidl.*`…). Liste et empreintes : relevé
`stock_extract_vendor.sh` (debugfs sur le `system.img` stock, lecture seule) ; même liste que `proprietary-files.txt`
de l'arbre. Autre firmware que V1.151R6 : `extract-files.sh ~/stock` reprend ces fichiers de votre `/system` (le jar,
épinglé, reste celui d'ici).

## Désodexage de autochips.jar

Le jar stock ne contient que son manifeste ; son code est un dex compact (`cdex001`) dans `boot-autochips.vdex`
(vdex 019). Refait comme `oat2dex()` d'`extract_utils.sh` (LineageOS 16.0), puis réécrit par d8 :

```bash
vdexExtractor -o out/ -i boot-autochips.vdex            # vendor/lineage/build/tools/Linux (sha256 e4c77a30…04cc)
compact_dex_converter out/boot-autochips_classes.cdex  # idem (sha256 9f74ccec…4771) -> dex 039
d8 --release --min-api 28 --output d8out classes.dex   # Android SDK build-tools 36.0.0
```

d8 est nécessaire : le dex de `compact_dex_converter` place un `annotation_set_item` sur une adresse non alignée
(`dexdump` : « Offset(393961) should be aligned by 4 for maplist item of type 4099 »), que le vérificateur d'ART
refuse. Le dex de d8 passe `dexdump -c` (somme et vérification) ; plus aucune instruction « quick ».
Le fichier est épinglé dans `proprietary-files.txt` de l'arbre (`-framework/autochips.jar|<sha1>`) : `extract-files.sh`
le garde au lieu de le refaire.

Empreintes : stock `autochips.jar` sha256 `364603c0…dec96`, `boot-autochips.vdex` sha256 `9fe6787f…cbef2d` ; jar
d'ici sha1 `18a76522…56fde3`.

Vérifié avant publication : aucune adresse MAC, numéro de série ni certificat (Apple MFi ou autre) dans ces fichiers.

## Licence

Fichiers propriétaires de Jancar / Autochips, repris de la ROM stock de l'appareil pour construire LineageOS sur ce
même appareil. Les makefiles générés sont sous Apache-2.0 (en-tête LineageOS).
