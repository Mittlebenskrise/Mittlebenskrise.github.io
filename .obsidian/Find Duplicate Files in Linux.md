One of the most straightforward tools to locate and remove duplicate files in folders is fdupes. It is an open-source and free duplicate file finder and was published on GitHub under the MIT License.

This Linux duplicate file finder uses a md5sum signature and byte-by-byte comparison verification to identify duplicate files in a directory. You can also do recursive searches, exclude specific search results, and display a list of duplicate files found if necessary.

With fdupes, you may either remove the duplicate files or replace them with links to the actual files after you have found them in a directory.
## Install Fdupes for Linux

```
sudo apt install fdupes
```

Find Duplicate Files in Linux With Fdupes
After the installation, you can use fdupes to find duplicate files.

Run the following command with your directory path to find duplicate files. This command only looks for duplicate files in the current folder. It does not search through subfolders and the like.

```
fdupes <directory path>
```

Run the fdupes command with the -r option to find duplicates throughout the folder and subfolders. The output shows that the "-r" option performs a more thorough search for duplicates in the folders and subfolders.

```
fdupes -r <directory path>
```

You can also look for duplicate files that aren't empty. It will allow you to concentrate on the task and eliminate the need to deal with empty files. Use the following command to enable this option.

```
fdupes -n <directory path>
```

To get more information on the set of duplicate files, use the fdupes command with the -m option.

```
fdupes -m <directory path>
```

You can also enter this fdupes command with the -S option to get duplicate file size information.

```
fdupes -S <directory path>
```

To save the outputs of the fdupes command, execute the following command.

```
fdupes <directory path> > output.txt
```

## Delete Duplicate Files in Linux With fdupes

Once the duplicates in the directory were narrowed down, use the fdupes command with the -d option to remove them.

```
fdupes -d <directory path>
```

remove duplicate files in linux with fdupes
You will be asked to save versions from the list of duplicate files. Enter the file number from the list to save the file.

Note: There are also advanced fdupes command options. Execute the fdupes commands with multiple options.

The following command will find all non-empty files in all folders and subfolders.

```
fdupes -n -r <directory path>
```

Or get an overview of all the duplicate files in the folders and subfolders by entering the following command.

```
fdupes -m -r <directory path>
```
 