---
sidebar: auto
---

# Screenlife Analytics

This is the development branch and the app might be buggy. Only merge to main once confirmed stable!

## Deployment guide

Ensure Docker is installed on your machine

Open terminal in the project's main directory
- To deploy: `docker compose -f docker-compose.dev.yml up --build --force-recreate`
	- `--build` flag can be dropped depends on the Docker version
- To stop containers:
	- If inside the same terminal, simply stop the process (<kbd>Ctrl</kbd> + <kbd>C</kbd>), and wait for Docker to stop the containers
	- If not: Open terminal in main directory -> `docker compose down`
	- In a terminal anywhere else: `docker stop $(docker ps -a -q)`
- To remove containers:
	- `docker container prune`
	- Use the Docker UI

Refreshing the work environment: Note that removing containers alone will not remove the database. The MongoDB database is stored in a detached volume and not affected by `docker compose down`.
- To remove all data, after stopping and removing containers, `docker volume prune`

To nuke the whole work environment (remove all containers, volumes and images): `docker system prune --all --force`

## Usage guide

### Creating dataset

Admin can create an **empty** dataset, but cannot add images remotely into the dataset. **This is by design,** to ensure decrypted images are not floating around online.

To create a brand new dataset:

- Select **Create** on home screen and fill in the form. Categories can be ignored, as the webapp is designed around allowing users to add categories as they wish.
- After the dataset has been created, images must be securely transferred to the corresponding directory on the server machine. **Ideally, this would be done physically using portable drives**. 
- After files have been transferred, click on the dataset and press **Scan**. This will build the database for usage.
#### Assigning dataset to user(s)

Only admin can grant other users access to a dataset. To do so:

- On home screen, click the ellipsis on the dataset description --> **Share**
- Select the users to assign --> **Save**

### Categories

There are two types of categories (tags):

- Batch tags: meant for image level annotation only
- Annotation tag: meant for image segmentation only

Creating or uploading tag(s) in the Batch Tagging page will create Batch type tags. Similarly, creating/uploading tags in the object annotator screen will create Annotation type tags.

It is not advisable to delete tags from the **Categories** page. Please use the **Delete Category** functions available in the annotation screens.

These tags can share the same name and supercategory as long as they are of different types (batch vs annotation). Note they will be given different ids in the database

### Image level annotation

Opening a dataset will default lead to the image level annotation page (labelled **Batch tagging** on the navbar). Each user is to supply their own categories, as these categories will not be shared among users.

#### Categories (tags) management

- **Create category**: The manual option, where user manually inputs the category name and supercategory. For color, if it's not changed, it will randomly assign a color.
- **Delete category**: Remove one or more categories from the list of active categories in use. Note that the category will remain in the **Categories** tab; in the future, if the user creates another identical category (same name, supercategory, and type), it will inherits the old one's id. 
    - Removing a category will not remove the annotations already applied to the images, even though visual indicators would disappear. The annotations will be preserved in export.
- **Import tagset**: For adding multiple tags at once under one same supercategory. Users can follow the standardized format for tagsets to create their own.
    - Importing a tagset will automatically remove all the existing tags of the same type. As mentioned above, this will not erase the annotations already applied.
- **Export tagset**: To export all currently active tags.

#### How to do image level annotation

- First import or create some tags. These will appear on top of the tools.
- To annotate:
	- Select images to apply tag. You can drag across the images to select multiple continuous images. Hold <kbd>Shift</kbd> while dragging/clicking to make multiple selections.
	- After selection, click on the tag to apply.
- To remove annotation:
	- Check the **Remove tag** box.
	- Select images to remove a tag from, similar to above
	- Click on desired tag to remove the annotation from all selected images.

### Image segmentation 

Found under the **Object Annotation** tab. To annotate, click on desired image.

#### Categories (tags) management

Similar to above.

Important: Adding or importing tags from this screen will create *Annotation* type tags. These tags are separate from *Batch* type tags used in image level annotation. As mentioned, a batch and annotation tag may share the same name and supercategory.

#### How to do image segmentation

- Click the **+** plus sign next to desired tag. An empty annotation will appear, and the tool will switch to bounding box.
- Draw bounding box on image, or switch to polygon tool to draw polygon.
- Repeat as necessary.
	- If the annotations are too cluttered, hide some tags using the eye icon on the left of the tag label.
- To delete an annotation, click the trash can next to the annotation.

#### Shortcuts

These are also found in the **Shortcuts guide** in the annotation screen

|Key  |Action  |
|--|--|
| <kbd>S</kbd> | Selection tool (hand) |
| <kbd>Q</kbd> | Bounding box (rectangular selection) |
| <kbd>E</kbd> | Polygon tool |
| <kbd>A</kbd> | Save and move to previous image |
| <kbd>D</kbd> | Save and move to next image |
| <kbd>Backspace</kbd> | Remove currently selected annotation |
| <kbd>Space</kbd> | Add new annotation with currently selected tag |
| <kbd>Ctrl</kbd> + <kbd>S</kbd> | Manual save |

### Exporting annotations

-  From Batch tagging screen, click **Export COCO**
- Leave the categories selection blank to export all categories (Note: only categories created/imported by current user)
- Exporting as CSV is available

After clicking **Export**, wait a few seconds as it might take some time for especially large datasets.

Your exported annotations will be ready in the **Exports** tab. The export tab saves up to 10 latest exports by this user.