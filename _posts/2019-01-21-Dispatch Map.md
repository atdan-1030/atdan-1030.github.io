---
layout: post
title: QGIS Dispatch
image: 
---

First we created a TSV file for all years of the dispatch article using your following code after importing os and re and setting the source:


    lof = os.listdir(source)
    counter = 0 # general counter to keep track of the progress

    def generate(filter):
    
    #(DANIEL): The follwing line creates an empty dictionary that will contain the TGN
    #numbers as keys and their frequencies as values.
    topCountDictionary = {}

    print(filter)
    counter = 0
    #(DANIEL): I removed the counter, because basically it does nothing in these lines
    #of code I think.
    #(DANIEL):Alternatively one could integrate it in the process to keep trace
    #of the placenames without tgn adding an else condition to the two ifs later on.
    
    for f in lof:
        if f.startswith("dltext"): # fileName test        
            with open(source + f, "r", encoding="utf8") as f1:
                text = f1.read()

                text = text.replace("&amp;", "&")

                # try to find the date
                date = re.search(r'<date value="([\d-]+)"', text).group(1)
                #print(date)

                if date.startswith(filter):
                    for tg in re.findall(r"(tgn,\d+)", text):
                        tgn = tg.split(",")[1]
                        
                        #(DANIEL): The following code adds one to the tgn's frequency
                        #, the TGN being the key or, if it isn't in the dictionary yet,
                        #sets it's value to 1.
                        
                        if tgn in topCountDictionary:
                            topCountDictionary[tgn] += 1
                        else:
                            topCountDictionary[tgn]  = 1

                        #input(topCountDictionary)
                        
    #(DANIEL): The following code transforms our dic into a list, eliminating double counts
    
    top_TSV = []

    for k,v in topCountDictionary.items():
        val = "%09d\t%s" % (int(v/2), k)
        # `tgn,XXXXX` occurs twice for every tagged toponym
        # NB: unfortunately, there are some toponyms/placenames that are tagged,
        #     but do not have tgn identifiers --- these will be missed;
        #     # how can these be accounted for? 

        top_TSV.append(val)                    

    # saving
    header = "freq\ttgn\n"
    with open("dispatch_toponyms_%s.tsv" % filter, "w", encoding="utf8") as f9:
        f9.write(header+"\n".join(top_TSV))
    #print(counter)

    #generate("186") #(DANIEL): This will collect all data because all dates start with"186". #(DANIEL): Here wo could also collect data of certain months calling the function with 1861-02 for example.

    generate("186")
    
Explanation: Looping inside a function that can "filter" dates makes it possible to create different data according to years, months (and even days)

The second step was to create a tsv file with the coordinates. Therefore we used the following lines of code provided by you:

    def generateTGNdata(source):

    lof = os.listdir(source)
    
    #(DANIEL): Here we create two lists: one for the TGNs with Coordinates, another one
    #for the ones without since we don't want to lose any data.
    tgnList = []
    tgnListNA = []
    count = 0

    for f in lof:
        if f.startswith("TGN"): # fileName test
            print(f)
            with open(source+f, "r", encoding="utf8") as f1:
                data = f1.read()

                data = re.split("</Subject>", data)

                for d in data:
                    d = re.sub("\n +", "", d)
                    #print(d)

                    if "Subject_ID" in d:
                        # SUBJECT ID
                        placeID = re.search(r"Subject_ID=\"(\d+)\"", d).group(1)
                        #print(placeID)

                        # NAME OF THE PLACE
                        placeName = re.search(r"<Term_Text>([^<]+)</Term_Text>", d).group(1)
                        #print(placeName)

                        # COORDINATES
                        if "<Coordinates>" in d:
                            latGr = re.search(r"<Latitude>(.*)</Latitude>", d).group(1)
                            lat = re.search(r"<Decimal>(.*)</Decimal>", latGr).group(1)

                            lonGr = re.search(r"<Longitude>(.*)</Longitude>", d).group(1)
                            lon = re.search(r"<Decimal>(.*)</Decimal>", lonGr).group(1)
                            #print(lat)
                            #print(lon)
                        else:
                            lat = "NA"
                            lon = "NA"

                        tgnList.append("\t".join([placeID, placeName, lat, lon]))
                        #input(tgnList)

                        if lat == "NA":
                            print("\t"+ "; ".join([placeID, placeName, lat, lon]))
                            tgnListNA.append("\t".join([placeID, placeName, lat, lon]))

    # saving
    header = "tgnID\tplacename\tlat\tlon\n"

    with open("tgn_data_light.tsv", "w", encoding="utf8") as f9a:
         f9a.write(header+"\n".join(tgnList))

    with open("tgn_data_light_NA.tsv", "w", encoding="utf8") as f9b:
         f9b.write(header+"\n".join(tgnListNA))

    print("TGN has %d items" % len(tgnList))

    generateTGNdata(source)
    
This code is easy to describe: We looped through the XML-files which content we defined as the variable "data". Then we split data on each <Subject> Tag and removed some redundant characters such as new lines and empty spaces. Then we collected the Subject_ID (being the TGN#), the Placename and the Coordinates of each place. If coordinates could'nt be attributed to the places we added them to the tgnListNA. Finally we saved the results in a TSV-file, making sure to save each ID into a new line by using "\n".join

Finally we matched the data. In order to do this, we first had to import re, os and create a funtion that converts our TSV file with the coordinates into a dictionary.

    def loadTGN(tgnTSV):
        with open(tgnTSV, "r", encoding="utf8") as f1:
          data = f1.read().split("\n")

          dic = {}
          #(DANIEL): Splitting on each tab because we use a TSV file we pour the data
          #from the TSV file into a dictionary called dic.
          for d in data:
            d = d.split("\t")

            dic[d[0]] = d

        return(dic)

    def match(freqFile, dicToMatch):
        with open(freqFile, "r", encoding="utf8") as f1:
          data = f1.read().split("\n") 
          #(DANIEL): We might want to call this data2 because data is allready defined above?
          #Nevertheless is worked out fine

          dataNew = []
          dataNewNA = []
          count = 0
          
          #(DANIEL): I think the following lines of code split the content of the TSV file
          #into tgnID and freq by distinguishing them through Tab1[1] and Tab0[0]
          for d in data[1:]:
            tgnID = d.split("\t")[1]
            freq  = d.split("\t")[0]
            
            #(DANIEL): This does the matching! If the tgnID from above is in the other
            #dictionary, we create a variable val which appends the frequency to the existing 
            #placename, TGN and Coords seperated by a tab.
            if tgnID in dicToMatch:
                val = "\t".join(dicToMatch[tgnID])
                val  = val + "\t" + freq
            
            #(DANIEL): We throw content that doesn't fit into the dataNewNA-List and the content that
            #fits into dataNew:
                if "\tNA\t" in val:
                    dataNewNA.append(val)
                else:
                    dataNew.append(val)
            else:
                print("%s (%d) not in TGN!" % (tgnID, int(freq)))
                count += 1
                

        header = "tgnID\tplacename\tlat\tlon\tfreq\n"

        with open("coord2_"+freqFile, "w", encoding="utf8") as f9a:
        f9a.write(header + "\n".join(dataNew))

        with open("coord2_NA_"+freqFile, "w", encoding="utf8") as f9b:
        f9b.write(header + "\n".join(dataNewNA))

        print("%d item have not been matched..." % count)

    dictionary = loadTGN("tgn_data_light.tsv")

    match("dispatch_toponyms_186.tsv", dictionary)
    
Finally we create a new TSV file that has the matched TGN, Name, Coords and the Freq as Appendix. We have to call the function match (which has two arguments) with our initially created TSV file with the placenames and TGNs from the dispatch and and compare it with "dictionary", which is the result of calling the function LoadTGN with the TSV from the XML files containing TGN, Name and Coordinates.

With the created TSV we could display the placenames and their frequencies in QGIS, creating a new layer and graduating the size of the dots by the value of freq. I only let higher frequencies be visible as you can see on my Screenshot: [QGIS Mapping2]

[QGIS Mapping2]: https://drive.google.com/open?id=1q31wKNpn63HBlmhQ_vpl2lemhnWvZ08t
