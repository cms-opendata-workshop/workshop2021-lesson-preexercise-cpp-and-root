---
title: "Using ROOT with C++ to write and read a file"
teaching: 20
exercises: 40
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

ROOT is a toolkit. That is, it is a set of functions and libraries that can be utilized in a variety
of languages and workflows. It was originally written in C++ and lends itself nicely to being used in standard,
compiled C++ code.

However, analysts wanted something more interactive, and so the ROOT team developed
[CINT, a C++ interpreter](https://root.cern.ch/root/html534/guides/users-guide/CINT.html). This gave users
an iteractive environment where they could type of C++ code one line at a time and have it executed
immediately. This gave rise to C++ scripts that many analysts use and in fact the sample
[ROOT tutorals](https://root.cern/doc/master/group__Tutorials.html)
are almost exclusively written as these C++ scripts (with a `.C` file extension). Because they are written
to run in CINT, they usually do not need the standard C++ `include` statements that you will see
in the examples below.

With the rise of the popularity of python, a set of Python-C++ bindings were written and eventually
officially supported by the ROOT development team, called [PyROOT](https://root.cern/manual/python/).
Many analysts currently write the code which plots or fits their code using PyROOT, and we will
show you some examples later in this exercise.

## What won't you learn here

ROOT is an incredibly powerful toolkit and has *a lot* in it. It is heavily used by most
nuclear and particle physics experiments running today. As such, a full overview is beyond the
scope of this minimal tutorial!

This tutorial will *not* teach you how to
* Make any plots more sophisticated than a basic histogram.
* Fit your data
* Use any of the HEP-specific libraries (e.g. `TLorentzVector`)

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
* [ROOT tutorial from Nevis Lab (Columbia Univ.)](https://www.nevis.columbia.edu/~seligman/root-class/). Very complete and always up-to-date tutorial from our friends at Columbia.

## ROOT terminology

To store these datasets, ROOT uses an object called `TTree` (ROOT objects are often prefixed by a `T`).

Each variable on the `TTree`, for example the transverse momentum of a muon, is stored in its own
`TBranch`.


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
We do this *before* we define the `TBranch`es in the `TTree`.

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

> ## Full source code file for `write_ROOT_file.cc`
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

Because we need to compile this in such a way that it links to the ROOT libraries, we will use a `Makefile`
to simplify the build process.

Create a new file called `Makefile` in the same directory as `write_ROOT_file.cc` and add the following to the
file. You'll most likely do this with the editor of your choice.

~~~
CC=g++

CFLAGS=-c -g -Wall `root-config --cflags`

LDFLAGS=`root-config --glibs`

all: write_ROOT_file

write_ROOT_file: write_ROOT_file.cc
    $(CC) $(CFLAGS) -o write_ROOT_file.o write_ROOT_file.cc
    $(CC) $(LDFLAGS) -o write_ROOT_file write_ROOT_file.o
~~~
{: .language-makefile}

> ## Warning! Tabs are important in Makefiles!
> Makefiles have been around a long time and are used for many projects, not just
> C/C++ code. While other build tools are slowly supplanting them (e.g. CMake),
> Makefiles are a pretty tried and true standard and it is worth taking time
> at some point and learning [more about them](https://www.tutorialspoint.com/makefile/makefile_rules.htm).
>
> One frustrating thing though can be a Makefile's reliance on *tabs* for specific purposes.
> In the example above, the following lines are preceeded by a *tab* and ***not*** four (4) spaces.
>
> ~~~
>    $(CC) $(CFLAGS) -o write_ROOT_file.o write_ROOT_file.cc
>    $(CC) $(LDFLAGS) -o write_ROOT_file write_ROOT_file.o
> ~~~
> {: .language-makefile}
> If your Makefile has spaces at those points instead of a tab, `make` will not work for you
> and you will get an error.
{: .callout}


You can now compile and run your compiled program from the command line!

~~~
make write_ROOT_file
./write_ROOT_file
~~~
{: language-bash}

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

> ## Will I have to `make` my Open Data analysis code?
> Yes, you will! However, you won't actually call `make`, nor will you need to write your own
> `Makefile`. Instead, the CMS software uses a configuration and build system called
> [SCRAM](https://twiki.cern.ch/twiki/bin/view/CMSPublic/SWGuideScram).
>
> So instead of
> typing `make`, you'll find yourself typing `scram`. However, it serves the same purpose
> by compiling and linking your code for you.
{: .callout}


## Read a ROOT file

Let's try to read this file in now. We won't do much with but we'll try to understand the process necessary
to read in all the data and loop over this event-by-event.

We'll start with the basic include statements and the main program.

~~~
#include<cstdio>
#include<cstdlib>
#include<iostream>

#include "TROOT.h"
#include "TTree.h"
#include "TFile.h"
#include "TRandom.h"

int main() {

    return 0;
}
~~~
{: .language-cpp}

In the `main` function, we'll define the input file.

~~~
   // Here's the input file
   // Without the 'recreate' argument, ROOT will assume this file exists to be read in.
   TFile f("tree.root");
~~~
{: .language-cpp}

We'll make use of the built-in member functions to `TFile` to pull out the `TTree` named
`t1`. There's a few other things to note.

First, we're going to assign it to a local variable named `input_tree`. This is to emphasize
that `t1` is just a string that refers to the name of the object stored in the `TFile` and
that we can assign it to any variable name, not just one named `t1`.

The second thing to note is that we are going to create a *pointer* to `input_tree`, which makes
some of the memory management easier. This means that we precede our variable name with
an asterix `*`, we have to cast the object pulled out of the `TFile` as a `TTree` pointer (`TTree*`),
and subsequent uses of `input_tree` will access data members and member functions with
the `->` operator rather than a period `.`.

If you want to learn more about pointers, there are [many](https://www.cplusplus.com/doc/tutorial/pointers/),
[many](https://www.w3schools.com/cpp/cpp_pointers.asp),
[resources](https://www.tutorialspoint.com/cplusplus/cpp_pointers.htm) out there.

~~~
   // We will now "Get" the tree from the file and assign it to
   // a new local variable.
   TTree *input_tree = (TTree*)f.Get("t1");
~~~
{: .language-cpp}

Just as we did in the `write_ROOT_file.cc` example, we will define some local variables.
These variables will actually get "filled" by the ROOT file when we loop over the events.

~~~
   Float_t met;
   Int_t njets;
   Float_t pt[16];
   Float_t eta[16];
   Float_t phi[16];
~~~
{: .language-cpp}

We'll now assign these local variables to specific `TBranch`es in `input_tree`. Note
that we'll be using the *address* of each local variable when we precede
the variable name with an ampersand `&`.

~~~
   // Assign these variables to specific branch addresses
   input_tree->SetBranchAddress("met",&met);
   input_tree->SetBranchAddress("njets",&njets);
   input_tree->SetBranchAddress("pt",&pt);
   input_tree->SetBranchAddress("eta",&eta);
   input_tree->SetBranchAddress("phi",&phi);

~~~
{: .language-cpp}

We're ready now to loop over events! Each time we call `input_tree->GetEntry(i)`,
it pulls the `i`th values out of `input_tree` and "fills" the local variables
with those values.

~~~
  for (Int_t i=0;i<nevents;i++) {

       // Get the values for the i`th event and fill all our local variables
       // that were assigned to TBranches
       input_tree->GetEntry(i);

       // Print the number of jets in this event
       printf("%d\n",njets);

       // Print out the momentum for each jet in this event
       for (Int_t j=0;j<njets;j++) {
           printf("%f,%f,%f\n",pt[j], eta[j], phi[j]);
       }
   }

~~~
{: .language-cpp}

The final version of your `read_ROOT_file.cc` should look like

> ## Full source code file for `read_ROOT_file.cc`
> ~~~
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
>   // Here's the input file
>   // Without the 'recreate' argument, ROOT will assume this file exists to be read in.
>   TFile f("tree.root");
>
>   // We will now "Get" the tree from the file and assign it to
>   // a new local variable.
>   TTree *input_tree = (TTree*)f.Get("t1");
>
>   Float_t met; // Missing energy in the transverse direction.
>
>   Int_t njets; // Necessary to keep track of the number of jets
>
>   // We'll define these assuming we will not write information for
>   // more than 16 jets. We'll have to check for this in the code otherwise
>   // it could crash!
>   Float_t pt[16];
>   Float_t eta[16];
>   Float_t phi[16];
>
>   // Assign these variables to specific branch addresses
>   input_tree->SetBranchAddress("met",&met);
>   input_tree->SetBranchAddress("njets",&njets);
>   input_tree->SetBranchAddress("pt",&pt);
>   input_tree->SetBranchAddress("eta",&eta);
>   input_tree->SetBranchAddress("phi",&phi);
>
>   // Get the number of events in the file
>   Int_t nevents = input_tree->GetEntries();
>
>   for (Int_t i=0;i<nevents;i++) {
>
>       // Get the values for the i`th event and fill all our local variables
>       // that were assigned to TBranches
>       input_tree->GetEntry(i);
>
>       // Print the number of jets in this event
>       printf("%d\n",njets);
>
>       // Print out the momentum for each jet in this event
>       for (Int_t j=0;j<njets;j++) {
>           printf("%f,%f,%f\n",pt[j], eta[j], phi[j]);
>       }
>   }
>
>   return 0;
> }
> ~~~
> {: .language-cpp}
{: .solution}

Now we need to modify our `Makefile` to compile this code.
We edit it so that it looks like this.

~~~
CC=g++

CFLAGS=-c -g -Wall `root-config --cflags`

LDFLAGS=`root-config --glibs`

all: write_ROOT_file read_ROOT_file

write_ROOT_file: write_ROOT_file.cc
    $(CC) $(CFLAGS) -o write_ROOT_file.o write_ROOT_file.cc
    $(CC) $(LDFLAGS) -o write_ROOT_file write_ROOT_file.o

read_ROOT_file: read_ROOT_file.cc
    $(CC) $(CFLAGS) -o read_ROOT_file.o read_ROOT_file.cc
    $(CC) $(LDFLAGS) -o read_ROOT_file read_ROOT_file.o

clean:
    rm -f ./*~ ./*.o ./write_ROOT_file
    rm -f ./*~ ./*.o ./read_ROOT_file
~~~
{: language-makefile}

We can now compile and run the code!

~~~
make read_ROOT_file
./read_ROOT_file
~~~
{: .language-bash}

We get a lot of output! However it should look something like the following, keeping
in mind your numbers will be different because of the random numbers that make up the values.

> ## Output of `read_ROOT_file`
> ~~~
> 1
> 85.105431,5.602912,0.501085
> 1
> 18.954712,4.375443,-1.546321
> 1
> 39.784435,5.165263,2.592412
> 3
> 80.748314,0.387768,1.786288
> 52.971573,3.939434,2.484405
> 12.969198,3.115963,0.910543
> 3
> 93.604256,0.737315,-0.647755
> 86.382034,3.493269,-1.573663
> 68.181541,3.658454,-1.206015
> 3
> 96.990395,5.839735,3.046098
> 79.096542,4.515290,0.039709
> 83.234497,4.990829,2.586360
> 4
> 60.880657,1.233623,-2.837789
> 25.723198,4.751074,2.355202
> 20.403908,4.656353,-2.171340
> 18.961079,1.425917,2.016828
> ~~~
> {: .output}
{: .solution}

Awesome! You've now written and read in a very simple ROOT file! There is obviously much more
that can be done, but this should give you the basics of interfacing with ROOT `TFile`s and
`TTree`s.

You'll see some version of this code when using *analyzers* to run over the open data code.
At that point, you can write out subsets of the data to new ROOT files or even simply dump the
data to a text or .csv file.

In the next section, we'll take a quick look at how to read in a file and make a few histograms,
still using the C++ syntax.

{% include links.md %}
