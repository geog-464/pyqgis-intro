# Basic programming with QGIS 3

Both ESRI products (ArcMap, ArcGIS Pro) and QGIS accommodate out-of-the-box python console interfaces for creating, loading and running scripts. Individual Python scripts can also be added as tools inside graphical modelers.

As you will see below, in software suites which require specific packages to interface with programmatically (i.e. the qgis.core module in QGIS), it's more about knowing the package than about necessarily just knowing Python. Knowing Python, however, will allow you to more or less infinitely extend and customize functionality beyond the available functions of any specific package. QGIS doesn't only allow you to create processing scripts: it also allows you to programmatically modify the user interface (e.g. dynamically hiding or displaying specific elements of the UI), user interactivity (via click [Actions](https://docs.qgis.org/3.22/en/docs/training_manual/create_vector_data/actions.html)), and map styles (via QGIS-specific [Expressions](https://docs.qgis.org/3.22/en/docs/user_manual/expressions/expression.html)). All QGIS objects and methods are found in the [official reference documentation](https://qgis.org/pyqgis/master/) (within which you could orient yourself using the [PyQGIS Cookbook](https://docs.qgis.org/3.22/en/docs/pyqgis_developer_cookbook/index.html) or additional open courseware linked to at the end of this tutorial).

## Importing data

Go to the Montreal Open Data portal and download the [réseau routier](https://donnees.montreal.ca/ville-de-montreal/geobase#resource-g%C3%A3%C2%A9obase). Whether you download a .shp or .geojson is up to you. If the latter, however, make sure to save as a .geojson and make sure to project it in MTM 8 before continuing.
- Launch QGIS and open the Plugins > Python Console.
- Click the Show Editor icon.
- Click the New Editor icon (+ button) and save it in your workspace as *import_vector.py*.
The simplest way to import a vector file into QGIS programmatically would be to enter the following:
```python
importPath = "PATH_TO_FILE\\FILENAME.shp"
iface.addVectorLayer(importPath,'LAYERNAME','ogr')
```
- Paste the above code and replace the PATH_TO_FILE and FILENAME appropriately. Replace LAYERNAME with a name of your choice to appear in your TOC. then run the script
  - Note that if on Windows, you will need to escape each backslash as seen in the example above (i.e. use two \\\\ instead of one wherever a slash appears in your PATH_TO_FILE).
  - If on Mac or Linux, a single forward slash (/) should suffice as a directory separator.

If this worked, great! But this isn't a portable script (i.e. it would be better if we didn't have to use absolute paths and use relative ones instead!).

- For asking the user to import a vector file no matter what computer the script is running from, you can use the following code instead (if on Mac, you can leave out the step that replaces / with \\)

```python
QgsProject.instance().setCrs(QgsCoordinateReferenceSystem(2950))
fileToImport = QInputDialog().getText(None, "Import vector file", "Enter path to file:")
fileToImport = fileToImport[0]
print(fileToImport)
#retrieve path as list so we can concatenate using windows slashes (\) instead
dirnameList = QgsProject.instance().fileName().split("/")[:-1]
#then concatenate by \
dirName = '\\'.join(dirnameList)+'\\'
importPath = dirName + fileToImport
print(importPath)
layer = iface.addVectorLayer(importPath,'LAYERNAME','ogr')
layer.renderer().symbol().setColor(QColor("darkGray"))
layer.triggerRepaint()
```

Notice the colour change once imported, which uses a [predefined color value](https://doc.qt.io/qt-5/qcolor.html#predefined-colors).

- Replace your previous code snippet with the the template above and study it to make sure you understand what each line is doing.
- Make the necessary modifications for the code to run properly on your machine and then run it.


## 2. Running geoprocessing scripts

- Search for and open Extract/clip by extent in the Processing Toolbox.
- Select your streets as the input layer and then use the Draw on canvas option to select an extent of your choice. Hit run.
- Review your output to make sure it makes sense, then remove it from your TOC.
- The code that was used to run the tool should be available in the Processing > History window. The log of your clip tool should look something like below:

```python
processing.run("native:extractbyextent",
{'INPUT':'PATH_TO_DATA/FILENAME.shp',
'EXTENT':'291035.789300000,297662.225800000,5035249.344900000,5039316.111100000 [EPSG:2950]',
'CLIP':True,
'OUTPUT':'TEMPORARY_OUTPUT'})
```

- Copy the code snippet from your history and paste it into a new, empty script. Give the OUTPUT value a filename with the .shp extension.
- Make sure PATH_TO_DATA and FILENAME make sense.
- Save the script as *clip_extent.py* and Run the script.

This should produce the same output as before, except this time, you might notice the file is located
elsewhere on your computer (in some root directory)... You might also notice it wasn't added to your map window. Why is this? What
variables and/or functions could we have included for this to be placed in our workspace and to be loaded into QGIS (see part 1…) ?

As you can see, processing scripts that import, display, modify, manipulate and build on data can be created in
QGIS and chained together in infinitely customizable ways. Scripts can help automate and optimize repetitive GIS
workflows, and can also be used for creating custom standalone tools and [publishing plugins](https://autogis-site.readthedocs.io/en/2019/lessons/L7/pyqgis.html#creating-qgis-plugins).

## Rendering Styles dynamically

In this tutorial we will apply some data-driven, dynamic styling to the road network to demonstrate how a
single layer can be programmatically styled for viewing within QGIS without any geoprocessing.

- Make sure the View > Panels > Layer Styling panel is available on the right-hand pane and make
sure your streets layer is selected.
- To modify the line widths:
  - Single Symbol > Simple Line > next to Stroke width, there should be an expression editor
dropdown > **Edit…**
  - Paste the following code:

```
case when "CLASSE" = '8' then @value*2.75 when 'CLASSE' = '7' then @value*2.25 when "CLASSE" = '6' then @value*1.75 when "CLASSE" = '5' then @value*1.5 end
```

What attributes are these modifications based on? How does line thickness reflect these attribute values?

- Switch to the Labels icon > Single Labels, and **Edit…** the Size parameter.
  - `case when "CLASSE" = '8' then @value*1.5 when "CLASSE" = '7' then @value*1.25 when "CLASSE" = '6' then @value*1.1 when "CLASSE" = '5' then @value*1 end`
- We can also pinpoint in what cases italics will apply: next to the I symbol > Edit…
  - `"CLASSE" = '8'`
- By then going to the Edit… window for *Value*, we can even filter what text is being used for our
street labels:
  - `case when "CLASSE" >= 5 AND "LIE_VOIE" IS NOT NULL AND "TYP_VOIE" IS NOT 'voie' then concat(title("TYP_VOIE"),' ',"LIE_VOIE",' ',"NOM_VOIE") when "CLASSE" >= 5 AND "LIE_VOIE" IS NULL AND "TYP_VOIE" IS NOT 'voie' then concat(title("TYP_VOIE"),' ',"NOM_VOIE") end`
- Under the Rendering tab on the far right, check off *Merge connected lines to avoid duplicate labels*.
- Under the Placement tab, set **Mode** to *Curved*.

You should now have a more cartographically-optimized street network than you had before! As you can see,
just about every styling parameter can be edited in this way. Try modifying the code you pasted in this
tutorial and using its logic to modify other parameters too!

## Additional resources

For those interested in further exploring optimizing and automating processes in QGIS, the free tutorials provided below are
strongly recommended:
- A [good walkthrough for beginners](https://anitagraser.com/pyqgis-101-introduction-to-qgis-python-programming-for-non-programmers/) made by the very active QGIS contributor Anita Graser
- A [very in-depth but well structured overview](https://courses.spatialthoughts.com/pyqgis-in-a-day.html) by QGIS titan Ujaval Gandhi
- A [module from an online course](https://autogis-site.readthedocs.io/en/2019/lessons/L7/pyqgis.html) on automating GIS workflows
- Another [overview](https://www.geodose.com/p/pyqgis.html) by a fairly active GIS blog
- A [video walkthrough](https://www.youtube.com/watch?v=h-mpUkwDdOQ) of QGIS expressions and variables by QGIS maintainer Nyall Dawson
