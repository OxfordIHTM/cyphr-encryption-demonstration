
<!-- README.md is generated from README.Rmd. Please edit that file -->

# Demonstration of an R workflow that implements encryption of shared/common files between collaborators

<!-- badges: start -->

<!-- badges: end -->

This repository demonstrates an example R workflow that implements
encryption of shared/common files between collaborators using the
`{cyphr}` package. This demonstration uses the vignette found
[here](https://docs.ropensci.org/cyphr/articles/data.html).

For this demonstration, the following scenario describes the use case of
the example project team.

## Scenario

A small project team of 4 people are collaborating on a research project
on cause of death (CoD) data. Given the nature of the data, the team’s
ethical responsibilities and commitments as per their respective
institution’s regulatory boards include ensuring that the raw CoD data
and all of its data derivatives are kept restricted only to authorised
research project team members. In addition, raw CoD data and all of its
data derivatives are kept encrypted when stored in each of the
authorised research project team members’ computers. When sharing the
data between each other, the research project team members need to
ensure that raw CoD data and all of its data derivatives are encrypted
on transit and can only be decrypted by authorised research project team
members.

Given this, the research project team need to devise a project workflow
that will satistfy the encryption requirements while at the same time
allowing access of the data to all authorised research project team
members. The research project team is using R or their data management,
analysis, and reporting and uses GitHub for versioning.

## Recommended/suggested workflow in R

The most appropriate tool in the R ecosystem that can support the
research project team in fulfilling the requirements for data protection
is the `{cyphr}` package\[1\].

Following is the recommended/suggested R workflow that will meet the
requirements for data protection as per the respective institution’s
regulations.

### Creation of personal SSH keys

The backbone of this recommended/suggested encryption workflow is the
use of personal Secure Shell (SSH) protocol keys. Each authorised
research project team member should create their personal SSH keys.

There are plenty of guidance available on the internet on how to do
this. This
[guide](https://docs.digitalocean.com/products/droplets/how-to/add-ssh-keys/create-with-openssh/)
is one of the most straightforward explanations on how to create your
personal SSH keys.

Best practice when generating your personal SSH keys is to always
generate a **passphrase** to encrypt your private key once it is
generated and stored in your computer. Without a passphrase, anyone that
can gain access to your computer will also be able use your personal SSH
keys.

Note that this step should be done on the *command line* or *terminal*
and not on R console.

### Create a key for the data and encrypt that key with your personal key

This step is a setup step that should be done by the administrator or
the research project team lead or any other research project team member
whose role it is to determine who has permissions to access the data.

Other members of the research project team will not need to perform this
step.

This step is done through the R console (directory or via an IDE i.e.,
RStudio) and is facilitated using the `{cyphr}` package (hence, the
`{cyphr}` package should be installed prior to doing these steps.

For this demonstration, we use a project repository structure where the
raw data will be placed within the `data-raw` directory and the
processed raw data will be stored within the `data` directory. So, we
will setup the key within the root directory of the project repository
for clarity and convenience when encrypting and decrypting files within
sub-directories.

To create a key, the following command should be issued in R:

``` r
cyphr::data_admin_init(".", path_user = path_key_admin)
```

where `path_key_admin` is the path to the personal SSH keys generated by
the admin or research project team lead. For purposes of this
demonstration, let us say that the admin or research project team lead
created their SSH key in the default `~.ssh/` directory. So, the command
above can be issued as follows instead:

``` r
cyphr::data_admin_init(".", path_user = "~.ssh/id_rsa")
```

or simply

``` r
cyphr::data_admin_init(".")
```

given that the `cyphr::data_admin_init()` function will use the default
SSH key path when no `path_user` is specified.

When running this command, the admin or the research project team lead
will be asked for the passphrase they created for their personal SSH key
(if they generated a passphrase). If the passphrase matches, then R will
generate a data key for the project repository and appropriately setup
the project for encryption. A directory named `.cyphr` will be created
in project root directory (since this is a hidden directory, select
*Show hidden files* in your file manager settings to see the directory).
This directory should be kept within the project repository and should
be committed to GitHub for versioning.

### Add encrypted data to the project repository

Now that the admin or the research project team lead has setup the
project repository for encryption, they can now add encrypted data to
the project.

For this demonstration, we will use the `iris` dataset as our example
raw data. We will store an ecrypted CSV copy of this dataset in the
`data-raw` directory using the following commands:

``` r
## Get the admin key ----
admin_key <- cyphr::data_key(".", path_user = path_key_admin)

## Store encrypted raw data in data-raw directory ----
cyphr::encrypt(
  expr = write.csv(iris, "data-raw/iris.csv", row.names = FALSE),
  key = admin_key
)
```

To check whether the `iris.csv` file in the `data-raw` directory is
indeed encrypted, we can try to read it:

``` r
read.csv("data-raw/iris.csv")
```

which results in the following error:

``` r
Error in make.names(col.names, unique = TRUE) : 
  invalid multibyte string at '<b4>/M+(<86><e1>9<99><aa><8b><cf>6A<f5>F~<bd><ff<9a><f5><92>t5<ef>{`<96><e0><92>iP<e1><bd>'
In addition: Warning messages:
1: In read.table(file = file, header = header, sep = sep, quote = quote,  :
  line 1 appears to contain embedded nulls
2: In read.table(file = file, header = header, sep = sep, quote = quote,  :
  line 3 appears to contain embedded nulls
```

But if we decrypt the file first and then read it:

``` r
cyphr::decrypt(
  expr = read.csv("data-raw/iris.csv"),
  key = admin_key
)
```

we are able to retrieve the data into R.

1.  <https://docs.ropensci.org/cyphr/>
