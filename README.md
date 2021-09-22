# Python-software-packaging

Here are some simple notes for packaging a reproducible python software in five different scenarios. 

*Warning*: this simplified tutorial is just a memo for my own use in the future so some steps, I admit, might be over-simplified. Thus, if you get stuck at some points please do not hesitate to post your problems in the Issues or contact me directly. 

### 1. Packaging for manually installation
If this was chosen the installation manner is usually:
~~~Bash
git clone package
cd package
python setup.py install
~~~
* Note: the environment has to be set well to make sure all dependencies can be used properly.

To pack a python software for such an installation manner, there are three aspects you should pay attention:

*1. the structure of the package (you can have a look at some examples below)*
> simple -> [panphlan](https://github.com/SegataLab/panphlan)
> medium -> [metaclock](https://github.com/SegataLab/metaclock)
> complex -> [phylophlan3](https://github.com/biobakery/phylophlan)

*2. the setup.py file configuration*
> [How to write the setup script](https://docs.python.org/3/distutils/setupscript.html)
>> e.g. [panphlan](https://github.com/SegataLab/panphlan/blob/master/setup.py)
     e.g. [metaclock](https://github.com/SegataLab/metaclock/blob/master/setup.py)
     e.g. [phylophlan](https://github.com/biobakery/phylophlan/blob/master/setup.py)

*3. the environment set-up for properly running the package*
  > Because nowadays conda is so trendy that I believe grabbing a good understanding of conda culture can solve 95% of your bioinformatics-related problems. If you need to revisit its knowledge please go to [managing conda environment](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#activating-an-environment)
 
### 2. Packaging for automatic installation using a private Conda channel

Such fashion of packaging is suitable for self-use or use for members within a group meanwhile pursuing an easy installation without much effort to make the package public. 

Installation manner is usually:

~~~Bash
conda create -n "env_package" -c private-channel package
~~~ 

For such fashion of packaging, besides handling well aspects related to package structure and writing setup script (please refer to **1. Packaging for manually installation**) we need to know a bit more how to config `meta.yml` (recipe for building a package) and building command lines.

*1. preparing meta.yml file*
 >Please refer to [How to define metadata (meta.yml)](https://docs.conda.io/projects/conda-build/en/latest/resources/define-metadata.html).
 >Some nice examples: [metaclock](https://github.com/SegataLab/metaclock/blob/master/recipe/meta.yaml), [phylophlan3](https://github.com/biobakery/phylophlan/blob/master/recipe/meta.yaml). 
 
 *2. build package*
 >Change directory into the one containing meta.yml file
 >~~~Bash
 >conda build .
 >~~~

*3. upload the built package to your own private channel*
>~~~Bash
>anaconda upload path-of-built-package
>~~~
>Note: after `conda build .` has been executed successfully it will prompt a link which infers to the path of built package. So just copy & paste the link.  
 
 
### 3. Packaging for installation using the bioconda channel

This packaging manner might be the most common way of sharing your mature open-source bioinformatics software if you want to reach out a wide user community. However, it is also quite a daunting task. Therefore, I will use more words in this section. 

First of first, it would be nice to re-visit [How to contribute to bioconda community](https://bioconda.github.io/contributor/index.html) before we continue to digest details. 

To contribute your package to bioconda community, the most challenging part is to design a metadata which can pass all required tests and same time it allows your package to work the way you want after installation. We are going cope all challenges block by block in a meta.yml file:
>1. *package*
>>name: the lower case name of the package. It may contain "-", but no spaces.
>>version: use the PEP-386 verlib conventions; use quotes (e.g. "1.4.1")
> 2. *source*
>> Specifies where the source code of the package is coming from. The source may come from a tarball file, git, hg, or svn. It may be a local path and it may contain patches. Here, we use the tarball file generated from GitHub tag.
>>> url: the link to tarball file of a released package (e.g. https://github.com/SegataLab/metaclock/archive/1.0.0.tar.gz )
>>> sha256: the checksum to verify the integrity of the source package.
>>>> ~~~Bash
>>>>wget -O- $URL | shasum -a 256
>>>>~~~
*Warning*: sha256 in the meta.yml recipe packed in the released software package might be different from the one in the meta.yml (should be the most updated one) sent to bioconda branch. But it does not affect anything about installation. Just bear in mind this small inconsistency due to practical updating reasons. 
>3. *build*
>> number: the build number should be incremented for new builds of the same version.
>> noarch: python
>>*"noarch" allows you to specify "no architecture" when building a package, thus making it compatible with all platforms and architectures. Noarch packages can be installed on any platform. For pure Python packages that can run on any Python version, you can use the noarch: python value.*
>> script: "{{ PYTHON }} -m pip install . --ignore-installed --no-deps -vv"
>> entry_points: Python entry points
>>> Entry points are used for calling some functions or objects in the comman line. If you are not familiar with entry points, please refer to [Python entry points](https://stackoverflow.com/questions/774824/explain-python-entry-points#:~:text=An%20%22entry%20point%22%20is%20typically,out%20in%20the%20comments!).
>4. *requirements*
>>host: Packages that need to be specific to the target platform, when the target platform is not necessarily the same as the build platform.
>> run: Packages required to run the package. These are the dependencies that are installed automatically whenever the package is installed (Try not to set too strict pins).
>5. *test*
>> commands: commands that are run as part of the test.
>6. *extra*
>> maintainers: GitHub handles of contributors
>7. *about*
>>home: GitHub link to the package repo
>>license: software package license
>>license_file: the name of license file
>>summary: a brief summary of the package

Once meta.yml file has been designed well, follow [here](https://bioconda.github.io/contributor/workflow.html) to complete the whole procedure.

### 4. Packaging for installation using PyPI

Packaging for PyPI is quite straightforward, please visit this nice [tutorial](https://medium.com/@joel.barmettler/how-to-upload-your-python-package-to-pypi-65edc5fe9c56) and you can get all knowledge you need. However, `pip install your-package`will not handle non-pip dependencies as well as conda, so if your package has many dependencies outside PyPi you need to carefully configure the environment when you install it in order to make the software can run properly.   

### 5. Updating and fixing bugs for packed scripts
Sometimes many problems pop up when a package is executed in a certain environment, so you might encounter some bugs after you packed your software and merged in some public repo. How should we efficiently fix the bugs and update our release? This can be quickly done in three steps:

1. Create an isolated conda environment where all dependencies that passed bioconda test are installed
2. Activate the environment, clone the repo and install the package using `python setup.py install`. 
3. Now package can run in this isolated environment and you can test it imitating the environment by bioconda channel
4. Fix bugs and re-submit to bioconda for review.  