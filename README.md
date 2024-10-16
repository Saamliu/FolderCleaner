[**Folder Cleaner**](https://resource.dopus.com/t/folder-cleaner-delete-folders-but-keep-files/52555) is a script suitable for [*Directory Opus*](https://www.gpsoft.com.au/). It can delete folders while keeping the files in them. You can also choose whether to recursively process subfolders.

🤔[What is Directory Opus?](https://www.iplaysoft.com/directory-opus.html)
***
![File structure | 690x409](images/File%20structure.png)
***
![Select folder | 690x409](images/Select%20folder.png)
***
![Execute script | 690x409](images/Execute%20script.png)
***
![Execution result | 690x409](images/Execution%20result.png)
***
**How to use**
* Method 1: Paste the script code directly

> Use the custom new button to edit the button, change the type of Function definition to Script Function, and the script type to JScript. Finally, paste the following script code in.

```
function OnClick(clickData) {
    var dlg = clickData.func.Dlg;
    dlg.message = "Are you sure you want to delete the selected folder but keep all files?";
    dlg.title = "Folder Delete Confirm";
    dlg.icon = "question";
    dlg.options(0).label = "Recursively process subfolders";
    dlg.options(0).state = true;
    dlg.options(1).label = "Show delete confirmation";
    dlg.options(1).state = true;
    dlg.buttons = "OK|Cancel";

    var result = dlg.Show();

    if (result === 1) {
        var cmd = clickData.func.command;
        var tab = clickData.func.sourcetab;
        var items = tab.selected_dirs;
        cmd.deselect = false; 

        for (var e = new Enumerator(items);!e.atEnd(); e.moveNext()) {
            var folder = e.item().realpath;
            if (dlg.options(0).state) {
                // If "Recursively process subfolders" is checked, perform recursive operation
                if (!moveFilesRecursively(folder, folder, dlg)) {
                    DOpus.Output("Error in recursive processing.");
                    break;
                }
            } else {
                // If not checked, perform non-recursive operation
                if (!moveFilesNonRecursively(folder)) {
                    DOpus.Output("Error in non-recursive processing.");
                    break;
                }
            }

            // Determine the parameters for delete command based on confirmation option
            var deleteParams = dlg.options(1).state? '' : 'QUIET';

            // Finally delete the original folder with appropriate parameters
            if (!cmd.RunCommand('Delete NORECYCLE SKIPNOTEMPTY FILE="' + folder + '" ' + deleteParams)) {
                DOpus.Output("Failed to delete folder.");
            }
        }
    } else {
        DOpus.Output("Operation canceled.");
    }
}

function moveFilesRecursively(currentFolderPath, rootFolderPath, dlg) {
    var fso = new ActiveXObject("Scripting.FileSystemObject");
    var folder = fso.GetFolder(currentFolderPath);
    var cmd = DOpus.Create.Command();

    // Iterate over all files in the current folder and move them to the parent directory of the root folder
    var files = new Enumerator(folder.Files);
    for (;!files.atEnd(); files.moveNext()) {
        var file = files.item();
        if (!cmd.RunCommand('Copy MOVE FILE="' + file.Path + '" TO="' + rootFolderPath + '\\.."')) {
            return false;
        }
    }

    // Recursively process subfolders
    var subfolders = new Enumerator(folder.SubFolders);
    for (;!subfolders.atEnd(); subfolders.moveNext()) {
        var subfolder = subfolders.item();
        if (!moveFilesRecursively(subfolder.Path, rootFolderPath, dlg)) {
            return false;
        }
    }

    // Delete the empty folder after processing subfolders if it is not the root folder
    if (currentFolderPath!== rootFolderPath) {
        var deleteParams = dlg.options(1).state? '' : 'QUIET';
        if (!cmd.RunCommand('Delete NORECYCLE SKIPNOTEMPTY FILE="' + currentFolderPath + '" ' + deleteParams)) {
            return false;
        }
    }

    return true;
}

function moveFilesNonRecursively(folderPath) {
    var fso = new ActiveXObject("Scripting.FileSystemObject");
    var folder = fso.GetFolder(folderPath);
    var cmd = DOpus.Create.Command();

    // Move all files in the current folder to the parent directory
    if (!cmd.RunCommand('Copy MOVE FILE="' + folderPath + '\\*" TO="' + folderPath + '\\.."')) {
        return false;
    }

    return true;
}
```

* Method 2: Install the script package

> Download the script package from the releases and run it directly. This will add a script (which can be configured and viewed through the Script Management) and a command to Directory Opus. Finally, drag the command to any location in the Customize > Commands interface (or other methods).