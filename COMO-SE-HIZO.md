# Cómo se hizo (y cómo repetirlo)

Guía para repetir exactamente el mismo proceso la próxima vez.

## 1. Montar la herramienta en el repositorio

El repositorio estaba vacío. Se copió el proyecto original (licencia MIT):

- Origen: https://github.com/guillaumemeyer/watermarks-remover
- Commit copiado: `1181fd4e8cc581931a5ee672697a646721e92c78` (2 oct 2026, versión v0.7.0)

```bash
git clone --depth 1 https://github.com/guillaumemeyer/watermarks-remover /tmp/upstream
(cd /tmp/upstream && tar --exclude=.git -cf - .) | tar -xf - -C .
git add -A && git commit -m "Importar watermarks-remover" && git push -u origin <rama>
```

Para actualizar a una versión más nueva: repetir lo mismo y hacer commit.

## 2. Quitar marcas a un archivo

Solo hace falta Python 3.10+ (sin dependencias).

```bash
# Ver qué marcas tiene
python3 service/scripts/inspect_file.py MI_ARCHIVO

# Limpiarlo (crea una copia limpia, no toca el original)
python3 service/scripts/clean_file.py MI_ARCHIVO -o MI_ARCHIVO.limpio.ext
```

Ejemplos:

```bash
python3 service/scripts/clean_file.py texto.md   -o texto.limpio.md
python3 service/scripts/clean_file.py foto.png   -o foto.limpia.png
python3 service/scripts/clean_file.py informe.docx -o informe.limpio.docx
python3 service/scripts/clean_file.py doc.pdf    -o doc.limpio.pdf
```

## 3. Qué quita y qué no

- **Texto:** caracteres invisibles (espacios de ancho cero, separadores invisibles,
  bidi, caracteres "tag") y espacios raros.
- **Archivos:** metadatos de procedencia C2PA / EXIF / XMP / propiedades del documento
  en imágenes, PDF, Office, audio y vídeo.
- **Marcas estadísticas de texto IA** (SynthID, etc.): no se detectan con scripts;
  hay que reescribir el texto (`service/scripts/rewrite_text.py`, ver README).
- **No quita marcas de agua visibles** (logos o texto encima de una imagen).
- Usar solo con contenido propio.

## 4. Prueba realizada (7 oct 2026)

Archivo de texto con `U+200B`, `U+202F` y `U+2063` ocultos:

- `inspect_file.py` → detectó 3 caracteres sospechosos.
- `clean_file.py` → `removed=2 replaced=1`; el resultado quedó solo con texto normal.

## 5. Para la próxima vez

1. Subir los archivos a limpiar a una carpeta (p. ej. `entrada/`).
2. Ejecutar `clean_file.py` sobre cada uno con salida en `salida/`.
3. Comprobar con `inspect_file.py` que la salida ya no tiene marcas.
4. Commit y push a la rama de trabajo.

## 6. Caso real: documentos de peritaje (14 oct 2026)

Archivos: 2 Word (`.docx`) y 1 Excel (`.xlsx`). Pasos seguidos:

1. Copiar los archivos subidos a `entrada/` (con nombres limpios, sin el prefijo de subida).
2. Inspeccionar: `python3 -I service/scripts/inspect_file.py entrada/X` → resultado:
   sin C2PA, sin metadatos de IA, sin marcas de agua visibles (revisado también
   cabeceras, pies y `comments.xml` a mano).
3. Limpiar: `python3 -I service/scripts/clean_file.py entrada/X -o salida/X`.
   Quitó: autor (`dc:creator`, `lastModifiedBy`), `Application` (LibreOffice),
   `custom.xml`, comentarios JPEG (`Lavc` = ffmpeg) y `tEXt` de PNG (Matplotlib).
4. Verificar:
   - `inspect_file.py` sobre la salida → C2PA False, AI metadata False.
   - `word/document.xml` idéntico antes/después (el texto no cambia).
   - Abrir con LibreOffice: `soffice --headless --convert-to pdf --outdir check salida/*`.
5. Entregar los archivos limpios por el chat. **No subirlos a GitHub** (son documentos
   privados del cliente), solo actualizar esta guía.

Ojo: la limpieza **borra el nombre del autor** de las propiedades del archivo. El texto
visible (cabeceras, pies, firma) no se toca.
