# Pandoc

## Generate doc file

### Command

``` PowerShell
pandoc "mdfile" -f markdown -t docx -s -o test.docx
```

### Generate Word document from md file on GitHub

Get Raw Url for md file using "Raw Button"

![GitHub Md Raw Button](.\media\GitHubMdRawButton.png)

**Exemple :**

``` PowerShell
pandoc "https://raw.githubusercontent.com/DynaFlo/D365FO-Scripts/master/Deployment/HostedBuild.md" -f markdown -t docx -s -o test.docx
```

## Links

<https://pandoc.org/installing.html>
