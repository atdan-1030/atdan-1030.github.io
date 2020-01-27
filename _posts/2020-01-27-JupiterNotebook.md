---
layout: post
title: Jupiter Notebook
---
In my effort to run the code via Jupiter Notebook, no error messages occured. However, the code seemed to have no effect and I couldn't find out where to change the code so that it actually did something (adding the right files somewhere?). 

So I just played around with parts of the code like the following:

%%time
 # Build LDA model
 number_of_topics = 30 # (Daniel) changed to 30
 lda_model = gensim.models.LdaModel(corpus=corpus,
                                    id2word=dictionary,
                                    num_topics=number_of_topics,
                                    update_every=20,
                                    passes=100,
                                    alpha='auto')
 print("-"*50)
 path = "./models_new/"
 lda_model.save(path+'dispatch_1864_neu30.lda')
 
 
 dispatchFile = "./dispatch/dispatch_1864.tsv" # or: dispatch_1864_filtered.tsv
dispatch = pandas.read_csv(dispatchFile, sep="\t", header=0)

# add a column with all dates of each month changed to 1 (we can use that to aggregate our data into months)
dispatch["month"] = [re.sub("-\d\d$", "-02", str(i)) for i in dispatch["date"]]

# (Daniel) here I changed the months by replacing 02 with 05...

# reorder columns
dispatch = dispatch[["id", "month", "date", "type", "header", "text"]]

print(dispatch["text"][1])

# (Daniel) changing text with date, header or type for example displays the selected type or the issue(?) in [].

I also hereby confirm the absolvement of the python course on codecademy.com
Here's the pictures: 



