<div class=figure>
  <p align="center"><img src="https://homepages.inf.ed.ac.uk/s1583634/paper_images/toy_tacl2018.png"
    width="450" height=auto></p>
  <p align="center"><small><i>Figure 1. Learning entailments that are consistent (A) across different but related typed entailment graphs and (B) within each graph. The dotted edges are missing, but will be recovered by considering relationships shown by across-graph (red) and within-graph (light blue) connections.</i></small></p>
</div>

This codebase contains the implementation for the following paper:

**Learning Typed Entailment Graphs with Global Soft Constraints**, *Mohammad Javad Hosseini, Nathanael Chambers, Siva Reddy, Xavier Holt, Shay B. Cohen, Mark Johnson, and Mark Steedman. Transactions of the Association for Computational Linguistics (TACL 2018).* [[paper]](https://www.mitpressjournals.org/doi/pdfplus/10.1162/tacl_a_00250)

All the data can be downloaded from the project's codalab page https://worksheets.codalab.org/worksheets/0x8684ad8e95e24c4d80074278bce37ba4/, except specified otherwise.


### How to Run the Code

Please follow the below instructions to create entailment graphs and/or replicate the paper's experiments.


**Step 1**: Clone the entGraph project.

**Step 2**: Download (and decompress) lib, lib_data and data folders inside the entGraph folder.

**Step 3**: Compile the Java files. In this step, it's assumed a remote machine (server) is used to run the code and a local machine (PC or laptop) is used to compile the code. One way would be to create a project using eclipse in the local machine. To do so, change the workspce directory to the folder containing entGraph. Then, create a project named entGraph (File -> New -> Java Project). From step 2, only the lib folder is needed for compilation. Then, the automatically created bin folder needs to be copied to the server that the code will be executed on. The rest will be done on the remote machine.

**Step 4**: You can simply download the linked and parsed NewsSpike corpus (NewsSpike_CCG_parsed.tar.gz) to your preferred location and skip to step 5. For more information on the parsing format, please see parsing_readme.txt. Alternatively, follow steps 4.1 to 4.5 to parse and link the NewsSpike corpus (or your own corpus) into predicate argument structure using the graph-parser (developped by Siva Reddy) based on CCG parser (easyCCG).

**Step 4.1**: Download the NewsSpike Corpus from http://www.cs.washington.edu/ai/clzhang/release.tar.gz and copy the data folder inside entGraph. Or make sure that your own corpus has the format that is required. Best place to see that is to look at one file from the input to the German pipeline. 
   
**Step 4.2**: I changed entailment.Util.convertReleaseToRawJson(inputJsonAddress) so that it points to my text. Shall this ever be released to a wider public I will change it to take command line input. It will print the output to news_raw.json. Run the code as:

    entailment.Util 

**Step 4.3**: Extract binary relations from the input json file: Javad recommends running the bash script prArgs.sh but I couldn't get that to work with my setup. Moreover the output obscures anything going wrong which led to me running this scrip for 24 hours with it actually just spewing errors that I didn't get to see, because the output is just numbers conting up. So, don't do that. Make sure that your input file is still named news_raw.json. 

The number of threads (numThreads) is a parameter which might need to be changed in constants.ConstantsParsing. Please keep the other parameters unchanged.

Run:
      
     entailment.LinesHandler


**Step 4.4**: Download news_linked.json and put it in folder aida. This is the output of NE linking (In our experiments, we used AIDALight). If you're using your own data, this is the point at which you get a version of Aida light and do the linking yourself. Enjoy. 

**Step 4.5**: Run entailment.Util (change the main function to run convertPredArgsToJson) as follows: 

    java -Xmx100G -cp lib/*:bin entailment.Util predArgs_gen.txt true true 12000000 aida/news_linked.json 1>news_gen.json 2>err.txt &

Input/output example argument are:

* predArgs_gen.txt: output of step 4.3.
* aida/news_linked.json: output of step 4.4.
* 120000000: is an upper bound on the number of lines of the corpus (this might need to be changed for a new corpus).     
* news_gen.json: It contains the linked and parsed NewsSpike corpus.

For larger corpora, instead of convertPredArgsToJson, you can use convertPredArgsToJsonUnsorted which will get less memory, but the output isn't sorted (this doesn't change any of the results for this paper).

**Step 5**: Extract the interim outputs:

You might need to set a few parameters in constants.ConstantsAgg:

* minArgPairForPred is C_1 in the paper, which is set to 3 by default.

* minPredForArgPair is C_2 in the paper, which is set to 3 by default.

* relAddress is the output of step 4.

* simsFolder is where the final output will be stored.

You need to run the entailment.vector.EntailGraphFactoryAggregator using:

    java -Xmx100G -cp lib/*:bin entailment.vector.EntailGraphFactoryAggregator

**Step 6**: The global learning.

A few parameters might need to be set in constants.ConstantsGraphs as follows:

* featName is the feature name to be used, which is by default BINC score.
* root is the folder address storing the output of step 5.

A few more parameters in constants.ConstantsSoftConst:

* numThreads, which I set that to 60 for a machine with 20 cpus, because not all the threads will run together. But you might need to change it.
* numIters is the number of iterations. lambda, lambda_2 and tau are set by default for Cross-Graph + Paraphrase-Resolution global soft constraints experiments, but can be tuned for another dataset.
* tPropSuffix, is the suffix of the new files that store the global scores.

Run:

    java -Xmx220G -cp lib/*:bin graph.softConst.TypePropagateMN.
  
**Step 7**: Please follow the instructions outlined in https://github.com/mjhosseini/entgraph_eval to evaluate the graphs on the entailment datasets.
  


### Download Learned Entailment Graphs

**Global Entailment Scores**: The globally consistent entailment scores are in "global_graphs.tar.gz".

For each type pair, like person#location, there is a file person#location_sim.txt which has the predicate similarities. For each predicate, its local and global similarities to other predicates are listed.



### Citation

If you found this codebase useful, please cite:

    @article{javad2018learning,
      title={Learning typed entailment graphs with global soft constraints},
      author={Javad Hosseini, Mohammad and Chambers, Nathanael and Reddy, Siva and Holt, Xavier R and Cohen, Shay B and       Johnson,  Mark and Steedman, Mark},
      journal={Transactions of the Association for Computational Linguistics},
      volume={6},
      pages={703--717},
      year={2018},
      publisher={MIT Press}
    }


