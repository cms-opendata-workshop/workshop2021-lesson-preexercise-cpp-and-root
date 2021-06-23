---
title: "Using ROOT with C++"
teaching: 0
exercises: 0
questions:
- "Why do I need to use ROOT?"
- "How do I use ROOT with C++?"
objectives:
- "Write a ROOT file using compiled C++ code."
- "Read a ROOT file using compiled C++ code."
keypoints:
- "ROOT defines the file format in which all of the CMS Open Data is stored."
- "These files can be accessed quickly using C++ code and the relevant information can be dumped out into other formats."
---

> ## Why ROOT?
>
> HEP data can be challenging! Not just to analyze but to store! The data don't lend themselves to 
> neat rows in a spreadsheet. One event might have 3 muons and the next event might have none. 
> One event might have 2 jets and the next event might have 20. What to do???
>
> The ROOT toolkit provides a file format that can allow for efficient storage of this type of
> data with heterogenous entries in each event. It *also* provides a pretty complete analysis
> environment with specialized libraries and visualization packages. Until recently, you had 
> to install the entire ROOT package just to read a file. The software provided by CMS to 
> read the open data relies on some minimal knowledge of ROOT to access. From there, you 
> can write out more ROOT files for further analysis or dump the data (or some subset of the data)
> to a format of your choosing. 
>
{: .testimonial}

## Interfacing with ROOT

C++, python/PyROOT, CINT

## What won't you learn here

ROOT is an incredibly powerful toolkit and has *a lot* in it. It is heavily used by most 
nuclear and particle physics experiments running today. As such, a full overview is beyond the
scope of this minimal tutorial! 

This tutorial will *not* teach you how to 
* Make any plots more sophisticated than a basic histogram.
* Fit your data
* Use any of the HEP-specific libraries (e.g. **TLorentzVector**)

## OK, where *can* I learn that stuff?

There are some great resources and tutorials out there for going further. 

* [The official ROOT Primer](https://root.cern/primer/). The recommended starting point to learn what ROOT can do. 
* [The official ROOT tutorials](https://root.cern/doc/master/group__Tutorials.html) This is a fairly comprehensive listing
of well-commented examples, written in C++ *scripts* that are designed to be run from within the ROOT C-interpreter. 
* [ROOT tutorial for Summer Students (2015).](https://indico.cern.ch/event/395198/). With video recordings!
* [Efficient analysis with ROOT](https://cms-opendata-workshop.github.io/workshop-lesson-root/). This is a more complete, 
end-to-end tutorial on using ROOT in a CMS analysis workflow. It was created in 2020 by some of our CMS colleagues 
for a separate workshop, but much of the material is relevant for the Open Data effort. It takes about 2.5 hours
to complete the tutorial. 

## ROOT terminology

To store these datasets, ROOT uses an object called **TTree** (ROOT objects are often prefixed by a **T**). 

Each variable on the **TTree**, for example the transverse momentum of a muon, is stored in its own
**TBranch**. 


## Write to a file

Let's open a file using our preferred editor. We'll call this file `write_ROOT_file.cc`. If we're
using `vi` as our editor, we would type 

~~~
vi write_ROOT_file.cc
~~~
{: .language-bash}

As in our last example, we first `include` some header files, both the standard C++ ones
and some new ROOT-specific ones.

~~~
#include<cstdio>
#include<cstdlib>
#include<iostream>

#include "TROOT.h"
#include "TTree.h"
#include "TFile.h"
#include "TRandom.h"
~~~
{: .language-cpp}

Note the inclusion of `TRandom.h`, which we'll be using to generate some random data for our
test file.

Next, we'll create our `main` function and start it off by defining our ROOT
file object. We'll also include some explanatory comments, which in the C++ syntax
are preceded by two slashes, `//`.

~~~
int main() {

    // Create a ROOT file, f.
    // The first argument, "tree.root" is the name of the file.
    // The second argument, "recreate", will recreate the file, even if it already exists.
    TFile f("tree.root","recreate");

    return 0;
}
~~~
{: .language-cpp}

Now we define the `TTree` object which will hold all of our variables and the data they represent.

This line comes after the `TFile` creation, but before the `return 0` statement at the end of the
main function. Subsequent edits will also follow the previous edit but come before `return 0` statement.

~~~
    // A TTree object called t1.
    // The first argument is the name of the object as stored by ROOT.
    // The second argument is a short descriptor.
    TTree t1("t1","A simple Tree with simple variables");
~~~
{: .language-cpp}

For this example, 
we'll assume we're recording the missing transverse energy, which means
there is only one value recorded for each event. 

We'll also record the energy and momentum (transverse momentum, eta, phi)
for jets, where there could be between 0 and 5 jets in each event.

This means we will define some C++ variables that will be used in the program.
We do this *before* we define the **TBranch**es in the **TTree**.

When we define the variables, we use ROOT's `Float_t` and `Int_t` types, which 
are analogous to `float` and `int` but are less dependent on the underlying 
computer OS and architecture. 

~~~
   Float_t met; // Missing energy in the transverse direction.

   Int_t njets; // Necessary to keep track of the number of jets

   // We'll define these assuming we will not write information for 
   // more than 16 jets. We'll have to check for this in the code otherwise
   // it could crash! 
   Float_t pt[16];
   Float_t eta[16];
   Float_t phi[16];
~~~
{: .language-cpp}


We now define the `TBranch` for the `met` variable.

~~~
    // The first argument is ROOT's internal name of the variable.
    // The second argument is the *address* of the actual variable we defined above
    // The third argument defines the *type* of the variable to be stored, and the "F"
    // at the end signifies that this is a float 
    t1.Branch("met",&met,"met/F");
~~~
{: .language-cpp}

Next we define the `TBranch`es for each of the other variables, but the syntax is slightly different
as these are acting as arrays with a varying number of entries for each event.

~~~
    // First we define njets where the syntax is the same as before,
    // except we take care to identify this as an integer with the final
    // /I designation
    t1.Branch("njets",&njets,"njets/I");

    // We can now define the other variables, but we use a slightly different
    // syntax for the third argument to identify the variable that will be used
    // to count the number of entries per event
    t1.Branch("pt",&pt,"pt[njets]/F");
    t1.Branch("eta",&eta,"eta[njets]/F");
    t1.Branch("phi",&phi,"phi[njets]/F");
~~~
{: .language-cpp}

OK, we've defined where everything will be stored! Let's now generate 1000 mock events.

~~~

    Int_t nevents = 1000;

    for (Int_t i=0;i<nevents;i++) {

        // Generate random number between 10-60 (arbitrary)
        met = 50*gRandom->Rndm() + 10;

        // Generate random number between 0-5, inclusive
        njets = gRandom->Integer(6);

        for (Int_t j=0;j<njets;j++) {
            pt[j] =  100*gRandom->Rndm();
            eta[j] =  6*gRandom->Rndm();
            phi[j] =  6.28*gRandom->Rndm() - 3.14;
        }

        // After each event we need to *fill* the TTree
        t1.Fill();
    }

    // After we've run over all the events, we "change directory" (cd) to the file object
    // and write the tree to it. 
    // We can also print the tree, just as a visual identifier to ourselves that 
    // the program ran to completion.
    f.cd();
    t1.Write();
    t1.Print();
~~~
{: .language-cpp}

The full `write_ROOT_file.cc` should now look like this

> ## Full source code file
> ~~~
>
>
> #include<cstdio>
> #include<cstdlib>
> #include<iostream>
> 
> #include "TROOT.h"
> #include "TTree.h"
> #include "TFile.h"
> #include "TRandom.h"
> 
> int main() {
> 
>    // Create a ROOT file, f.
>    // The first argument, "tree.root" is the name of the file.
>    // The second argument, "recreate", will recreate the file, even if it already exists.
>    TFile f("tree.root","recreate");
>
>    // A TTree object called t1.
>    // The first argument is the name of the object as stored by ROOT.
>    // The second argument is a short descriptor.
>    TTree t1("t1","A simple Tree with simple variables");
>
>    Float_t met; // Missing energy in the transverse direction.
>
>    Int_t njets; // Necessary to keep track of the number of jets
>
>    // We'll define these assuming we will not write information for
>    // more than 16 jets. We'll have to check for this in the code otherwise
>    // it could crash!
>    Float_t pt[16];
>    Float_t eta[16];
>    Float_t phi[16];
>
>    // The first argument is ROOT's internal name of the variable.
>    // The second argument is the *address* of the actual variable we defined above
>    // The third argument defines the *type* of the variable to be stored, and the "F"
>    // at the end signifies that this is a float
>    t1.Branch("met",&met,"met/F");
>
>    // First we define njets where the syntax is the same as before,
>    // except we take care to identify this as an integer with the final
>    // /I designation
>    t1.Branch("njets",&njets,"njets/I");
>
>    // We can now define the other variables, but we use a slightly different
>    // syntax for the third argument to identify the variable that will be used
>    // to count the number of entries per event
>    t1.Branch("pt",&pt,"pt[njets]/F");
>    t1.Branch("eta",&eta,"eta[njets]/F");
>    t1.Branch("phi",&phi,"phi[njets]/F");
>
>    Int_t nevents = 1000;
>
>    for (Int_t i=0;i<nevents;i++) {
>
>        // Generate random number between 10-60 (arbitrary)
>        met = 50*gRandom->Rndm() + 10;
>
>        // Generate random number between 0-5, inclusive
>        njets = gRandom->Integer(6);
>
>        for (Int_t j=0;j<njets;j++) {
>            pt[j] =  100*gRandom->Rndm();
>            eta[j] =  6*gRandom->Rndm();
>            phi[j] =  6.28*gRandom->Rndm() - 3.14;
>        }
>
>        // After each event we need to *fill* the TTree
>        t1.Fill();
>    }
>
>    // After we've run over all the events, we "change directory" to the file object
>    // and write the tree to it.
>    // We can also print the tree, just as a visual identifier to ourselves that
>    // the program ran to completion.
>    f.cd();
>    t1.Write();
>    t1.Print();
>    
>    return 0;
> }
> ~~~
> {: .language-cpp}
{: .solution}

Because need to compile this in such a way that it links to the ROOT libraries, we will use a `Makefile` 
to simplify the build process. 

Create a new file called `Makefile` in the same directory as `write_ROOT_file.cc` and add the following to the
file. You'll most likely do this with the editor of your choice. 

~~~
CC=g++

CFLAGS=-c -g -Wall `root-config --cflags`

LDFLAGS=`root-config --glibs`

SOURCES=write_ROOT_file.cc

OBJECTS=$(SOURCES:.cc=.o)
    EXECUTABLE=write_ROOT_file

all: $(SOURCES) $(EXECUTABLE)

$(EXECUTABLE): $(OBJECTS)
        $(CC) $(LDFLAGS) $(OBJECTS) -o $@

.cc.o:
        $(CC) $(CFLAGS) $< -o $@

clean:
        rm -f ./*~ ./*.o ./write_ROOT_file
~~~
{: .language-makefile}

WARNING ABOUT TABS!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

You can now run your compiled program from the command line!

~~~
./write_ROOT_file
~~~
{: language-bash}
~~~

> ## Output from `write_ROOT_file`
> ~~~
> ******************************************************************************
> *Tree    :t1        : A simple Tree with simple variables                    *
> *Entries :     1000 : Total =           51536 bytes  File  Size =      36858 *
> *        :          : Tree compression factor =   1.35                       *
> ******************************************************************************
> *Br    0 :met       : met/F                                                  *
> *Entries :     1000 : Total  Size=       4542 bytes  File Size  =       3641 *
> *Baskets :        1 : Basket Size=      32000 bytes  Compression=   1.12     *
> *............................................................................*
> *Br    1 :njets     : njets/I                                                *
> *Entries :     1000 : Total  Size=       4552 bytes  File Size  =        841 *
> *Baskets :        1 : Basket Size=      32000 bytes  Compression=   4.84     *
> *............................................................................*
> *Br    2 :pt        : pt[njets]/F                                            *
> *Entries :     1000 : Total  Size=      14084 bytes  File Size  =      10445 *
> *Baskets :        1 : Basket Size=      32000 bytes  Compression=   1.29     *
> *............................................................................*
> *Br    3 :eta       : eta[njets]/F                                           *
> *Entries :     1000 : Total  Size=      14089 bytes  File Size  =      10424 *
> *Baskets :        1 : Basket Size=      32000 bytes  Compression=   1.30     *
> *............................................................................*
> *Br    4 :phi       : phi[njets]/F                                           *
> *Entries :     1000 : Total  Size=      14089 bytes  File Size  =      10758 *
> *Baskets :        1 : Basket Size=      32000 bytes  Compression=   1.26     *
> *............................................................................*
> ~~~
> {: .output}
{: .solution}

Your numbers may be slightly different because of the random numbers that 
are generated.

Huzzah! You've successfully written your first ROOT file!

## Read a ROOT file








{% include links.md %}

