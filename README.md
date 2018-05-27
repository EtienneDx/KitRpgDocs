# KitRpgDocs
This package only contains the documentation for all oficial KitRpg packages

## Common behaviour documentation

### Dynamically formatted text

A text can be dynamically formated using brackets to replace variables, which can be of multiple types.

First possible usage is to replace some brackets by the string value of the variable : 

    {VarName}
    
Another usage is to replace by another text, depending on the value of the variable : 

    {VarName?This text will be shown if VarName is true:This one will be shown otherwise}
    
This will check wether the variable string value is literaly "true", and if it's the case, show the first option, or the second otherwise.

If you want to display something only if a certain variable is true, you can do it like this : 

    {VarName?This will be shown if VarName is true}
