============================
HLT JEC - Example
============================


About
=====

1. Compute environment
----------------------

In order to be able to rerun the analysis even several years in the future, we
need to "encapsulate the current compute environment", for example to freeze the
ROOT version our analysis is using. We shall achieve this by preparing a `Docker
<https://www.docker.com/>`_ container image for our analysis steps.

This analysis example runs within the `CMSSW <http://cms-sw.github.io/>`_
analysis framework that was packaged for Docker in `clelange/cmssw:10_6_12 <https://hub.docker.com/layers/clelange/cmssw/10_6_12/images/sha256-38378fdfdcc8f75a5c33792d67ca8f79ea90cccd0c0627bfb4e20ee7d37039ce?context=explore/>`_. The code found in the directory `<code/>`_ was added to the docker image with the `<Dockerfile>`_.

The necessary code to build the Docker image can be found at `github.com/alintulu/reana-demo-JetMETAnalysis <https://github.com/alintulu/reana-demo-JetMETAnalysis>`_.
Build the Docker image via the command line interface

.. code-block:: console

  git clone https://github.com/alintulu/reana-demo-JetMETAnalysis.git
  cd reana-demo-JetMETAnalysis
  docker build -t alintulu/cmssw:10_6_12 .

2. Analysis workflow
--------------------

This worfklow could in theory run in serial, however to speed up the process an save memory most of the steps are scattered and ran in parallel. We use the `Yadage <https://github.com/yadage>`_ workflow engine to
express the computational steps in a declarative manner. The `workflow.yaml <workflow/workflow.yaml>`_ workflow defines the full pipeline.

.. code-block:: console

   +-------------------+
   | Ntuple production |   Running parallel
   +-------------------+
      |      |      |    
     +----+  |  +------+
     | PU |  |  | NoPU |
     +----+  |  +------+   
      |      |      |
      v      v      v
   +-------------------+
   | Produce histograms|   Running parallel
   +-------------------+
     |      |      |    
     |      |      |
     v      v      v  <-- Merge            
    +--------------+   
    | Compute L2L3 |   Single process
    +--------------+
           |
           |                             
           v                                                           
   +-----------------------+
   | Compute Closure files |   Running parallel
   +-----------------------+
     |      |      |    
     |      |      |
     v      v      v   <-- Merge
   +---------------------+
   |  Draw Closure plots |   Single process
   +---------------------+
           |
           |
           v
         DONE

At a very high level the workflow is as follows:

Prepare input:
  1. Create Ntuples by extracting the important information from data files.

L2Relative and L3Absolute corrections:
  1. Compute L2 relative energy correction and L3 absolute response in the barrel.
  
Closure:
  1. Apply L2L3 corrections.
  2. Compute closure plots to test the validity of the implemented methods. The closure is shown in jet response vs pT and jet response vs eta. The jet response is defined as the average value of the ratio of measured jet pT to particle level jet pT. A value close to one indicates a succesfull computation of the jet energy corrections.

Running the example on REANA cloud
==================================

We start by creating a `reana.yaml <reana.yaml>`_ file describing the above
analysis structure with its inputs, code, runtime environment, computational
workflow steps and expected outputs. In this example we are using the Yadage
workflow specification, with its steps in the `workflow <workflow>`_ directory.


.. code-block:: yaml

    version: 0.6.0
    inputs:
      directories:
        - workflow
    workflow:
      type: yadage
      file: workflow/workflow.yaml

We can now install the REANA command-line client, run the analysis and download the resulting plots:

.. code-block:: console

    $ # create new virtual environment
    $ virtualenv ~/.virtualenvs/myreana
    $ source ~/.virtualenvs/myreana/bin/activate
    $ # install REANA client
    $ pip install reana-client
    $ # connect to some REANA cloud instance
    $ export REANA_SERVER_URL=https://reana.cern.ch/
    $ export REANA_ACCESS_TOKEN=XXXXXXX
    $ # create new workflow
    $ reana-client create -n my-analysis
    $ export REANA_WORKON=my-analysis
    $ # upload input code and data to the workspace
    $ reana-client upload 
    $ # start computational workflow
    $ reana-client start
    $ # ... should be finished in about 15 minutes
    $ reana-client status
    $ # list output files
    $ reana-client ls

Please see the `REANA-Client <https://reana-client.readthedocs.io/>`_
documentation for more detailed explanation of typical ``reana-client`` usage
scenarios.

Contributors
============

The list of contributors in alphabetical order:

- `Adelina Lintuluoto <https://orcid.org/0000-0002-0726-1452>`_
