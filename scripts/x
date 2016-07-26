
# coding: utf-8
Script for counting the number of times an animal was photographed, counting the number of contributor per photograph,
    average pics per contributor and for creating csv files and ploting the contributor/feature 
Author: Sreejith Menon
Date: May 25 2016

Modified: Anthony Leon and Jose Hernandez
Date: July 14 2016 

import GetPropertiesAPI as GP
from collections import Counter
import pandas as pd
import importlib
importlib.reload(GP)
import statistics as s
import operator
from collections import defaultdict
import csv 
import matplotlib.pyplot as plt
import numpy as np


aid_list = []
gidAidDict = {}
for gid in range(1,9407):
    aid = GP.getAnnotID(gid)
    aid_list.append(aid)
    gidAidDict[gid] = aid

aid_list = list(filter(lambda x: x != None,aid_list))

aids = []
for aid_l in aid_list:
    for aid in aid_l:
        aids.append(aid)


aidContribTupList = []
for aid in aids:
    contrib = GP.getImageFeature(aid,'image_contributor_tag')
    aidContribTupList.append((aid,contrib[0]))


aidNidMap = {}
aidNamesMap = {}
aidNidTupList = [] # modified

for aid in aids:
    nid = GP.getImageFeature(aid,'nids')
    aidNidMap[aid] = nid
    aidNidTupList.append((aid,nid[0]))

nids = []
for aid in aidNidMap.keys():
    nids.append(aidNidMap[aid][0])

nids = list(filter(lambda x : x > 0,nids))

counter_nid= Counter(nids)

gidAidDict
aidGidTupList = [] # key : aid and value : gid # modified
for gid in gidAidDict.keys():
    if gidAidDict[gid] != None:
        for aid in gidAidDict[gid]:
            aidGidTupList.append((aid,gid))

aidGidDf = pd.DataFrame(aidGidTupList,columns = ['AID','GID']) 
aidNidDf = pd.DataFrame(aidNidTupList,columns = ['AID','NID']) 
aidContribDf = pd.DataFrame(aidContribTupList,columns = ['AID','CONTRIBUTOR'])
aidNidDf = aidNidDf[(aidNidDf['NID']>0)]

aidGidNidDf = pd.merge(aidGidDf,aidNidDf,left_on = 'AID',right_on = 'AID')
aidGidNidContribDf = pd.merge(aidGidNidDf,aidContribDf,left_on = 'AID',right_on = 'AID')
aidGidNidContribDf.to_csv('results.csv',index=False)


# Arguments : CSV file, boolean(True create csv files, and plot, False return Dictionary)
# Returns: CSV files, plot or Dictionary(key : CONTRIBUTOR and value: Total photos taken)          
def picsTakenByContributor(filename, boolean):
    with open('results.csv') as f: # read from csv file into a key : GID and value : CONTRIBUTOR
        reader = csv.DictReader(f)
        gidContribMap = { line['GID']: line['CONTRIBUTOR'] for line in reader }
    
    ContribTotal = {} # dict with key : CONTRIBUTOR and value: Total photos taken

    for gid,contrib in gidContribMap.items():
        ContribTotal[contrib] = ContribTotal.get(contrib,0) + 1
    
    if boolean == True:
        csv_out = csv.writer(open('ContributorTotal.csv', 'w')) 
        csv_out.writerow(['CONTRIBUTOR', 'TOTAL'])
        for Contrib, value in ContribTotal.items():
            csv_out.writerow([Contrib, value])
            
        data = pd.read_csv('ContributorTotal.csv', sep=',',header=0, index_col =0)

        data = data.sort_values('TOTAL', ascending=False)

        data.plot(kind='bar')
        plt.ylabel('Number of pictures taken')
        plt.xlabel('Contributor')
        plt.title('Total Images/Contributor Totals')

        #plt.show()
        plt.savefig(str("contributorTotal.png") ,bbox_inches='tight')
    else:
        print(s.mean(ContribTotal.values()))
        print(s.stdev(ContribTotal.values()))
        return ContribTotal
    
    return


# Arguments : CSV file, boolean(True create csv files, False return Dictionary)
# Returns: CSV file or Dictionary(key : NID, CONTRIBUTOR and value: Total photos taken)   
def numberPicsIndividualContributor(filename, boolean):

    with open('results.csv') as f2: # read from csv file into a Dict with key : AID and value : GID, NID, CONTRIBUTOR
        reader2 = csv.DictReader(f2)
        aidToGidNidContribMap = { line['AID']: [line['GID'], line['NID'], line['CONTRIBUTOR']] for line in reader2 }
    
    NidContribTotal = {} # dict with key : NID, CONTRIBUTOR and value: Total photos taken
    for aid,(gid,nid,contrib) in aidToGidNidContribMap.items():
         NidContribTotal[nid, contrib] = NidContribTotal.get((nid,contrib),0) + 1
            
    if boolean == True:
        csv_out = csv.writer(open('nidtoContributor.csv', 'w')) 
        csv_out.writerow(['NID', 'CONTRIBUTOR', 'TOTAL'])
        for (Nid, Contrib), value in NidContribTotal.items():
            csv_out.writerow([Nid, Contrib, value])
            
    else:
        return NidContribTotal
    
    return


# Arguments : CSV file, boolean(True create csv files, False return Dictionary)
# Returns: CSV file or Dictionary(key : CONTRIBUTOR and value: Average photos taken)  
def averagePicsTakenByContributor(filename, boolean):
    NidContribTotal=numberPicsIndividualContributor(filename, False)

    totalPicsPerContrib = {}
    uniqueAnimalsPerContrib = {}
    for key in NidContribTotal:
        totalPicsPerContrib[key[1]] = totalPicsPerContrib.get(key[1], 0) + NidContribTotal[key]
        uniqueAnimalsPerContrib[key[1]]=uniqueAnimalsPerContrib.get(key[1], 0) + 1
    
    averagePicsPerContrib={}
    for key in totalPicsPerContrib:
        averagePicsPerContrib[key]=totalPicsPerContrib[key]/uniqueAnimalsPerContrib[key]
   
    if boolean == True:
        csv_out = csv.writer(open('averagePicsTakenByContributor.csv', 'w')) 
        csv_out.writerow(['CONTRIBUTOR', "Average"])
        for contrib, average in averagePicsPerContrib.items(): 
            csv_out.writerow([contrib, average])
    
    else:
        return averagePicsPerContrib


# Arguments : CSV file, Required Feature, boolean
# Accepted Features: species_texts, age_months_est, exemplar_flags, sex_texts, yaw_texts, quality_texts,image_contributor_tag
#                    boolean = True if want individual csv files, False if only want end product
# Returns : None
# Creates: specific feature csv file and/or all feature csv file, all feature csv file with sum and plot
def getContributorFeature(filename, feature, boolean):
    
    with open(filename) as f: # read from csv file into a Dict with key : AID and value : GID, NID, CONTRIBUTOR
        reader = csv.DictReader(f)
        aidToGidNidContribMap = { line['AID']: [line['GID'], line['NID'], line['CONTRIBUTOR']] for line in reader }
    
    contribToFeatureMap = defaultdict(list) # dict where key : contributor and values : List of feature
    for aid,(gid,nid,contrib) in aidToGidNidContribMap.items():
        if feature != "age/months":
            contribToFeatureMap[contrib].append(GP.getImageFeature(aid, feature)[0])
        else:
            contribToFeatureMap[contrib].append(GP.getImageFeature(aid, feature))
    
    if feature == "age/months":
        contribToFeatureMap = dict(contribToFeatureMap)
        result = {}

        for contrib in contribToFeatureMap.keys():
             result[contrib] = [GP.getAgeFeatureReadableFmt(i) for i in contribToFeatureMap[contrib]]
        
        result2=defaultdict(list)
        for contrib, age_list in result.items():
            for age in age_list:
                for ind_age in age:
                    result2[contrib].append(ind_age)
        
        contribAnimFeatCount = {} # dict where key : contributor and values : total of specific feature
        for key in result2.keys():
            contribAnimFeatCount[key]=dict(Counter(result2[key]))
        
        if boolean == True:
            getContributorSpecificFeature(contribAnimFeatCount, 'juveniles- two year old')
            getContributorSpecificFeature(contribAnimFeatCount, 'juveniles - one year old')
            getContributorSpecificFeature(contribAnimFeatCount, 'infant')
            getContributorSpecificFeature(contribAnimFeatCount, 'adult')
            getContributorSpecificFeature(contribAnimFeatCount, 'unknown')
        
            createcombinedcsv(feature, 'juveniles- two year old', 'juveniles - one year old', 'infant', 'adult', 'unknown')
        
        else:
            juveniles_2 = getContributorSpecificFeatureList(contribAnimFeatCount, 'juveniles- two year old')
            juveniles_1 = getContributorSpecificFeatureList(contribAnimFeatCount, 'juveniles - one year old')
            infant = getContributorSpecificFeatureList(contribAnimFeatCount, 'infant')
            adult = getContributorSpecificFeatureList(contribAnimFeatCount, 'adult')
            unknown = getContributorSpecificFeatureList(contribAnimFeatCount, 'unknown')
            
            createCombinedCsvFromList(feature, "juveniles- two year old", 'juveniles - one year old', 'infant', 'adult', 'unknown', juveniles_2, juveniles_1, infant, adult, unknown)
            
        sumRows('juveniles- two year old-unknown.csv')
        
        createStackgraph("sumjuveniles- two year old-unknown.csv", "age/months", 'juveniles- two year old', 'juveniles - one year old', 'infant', 'adult', 'unknown')
        
        return # no need to continue after this

    
    # put this code here because has to do something different than the other ones
    if feature == "exemplar":
        exemplarDict=defaultdict(list)  #change 0 and 1 to 'zero' and 'one' so getContributorSpecificFeature method can work
        for contrib, value in contribToFeatureMap.items():  
            for ind in value:
                if ind == 0:
                    ind ='zero'
                else:
                    ind ='one'
                exemplarDict[contrib].append(ind)
            
        exemplarRegDict={}
        for key in exemplarDict.keys():
            exemplarRegDict[key]=dict(Counter(exemplarDict[key]))     
        
        if boolean == True:
            getContributorSpecificFeature(exemplarRegDict, 'zero')
            getContributorSpecificFeature(exemplarRegDict, 'one')
            
            createcombinedcsv(feature,"zero", "one")
        else:
            zero = getContributorSpecificFeatureList(exemplarRegDict, 'zero')
            one = getContributorSpecificFeatureList(exemplarRegDict, 'one')
            
            createCombinedCsvFromList(feature, 'zero', 'one', zero, one)
        
        sumRows('zeroandone.csv')
        
        createStackgraph("sumzeroandone.csv", "exemplar", "zero", "one")
        
        return # no need to continue after this
    
    #NOT "exemplar_flags" or 'age_months_est' so continue on
    
    contribAnimFeatCount = {} # dict where key : contributor and values : total of specific feature
    for key in contribToFeatureMap.keys():
        contribAnimFeatCount[key]=dict(Counter(contribToFeatureMap[key]))

    if feature == "sex/text":  
        # get each individual csv files where Contributor, SpecFeature if boolean is True
        if boolean == True:
            getContributorSpecificFeature(contribAnimFeatCount, 'Male')
            getContributorSpecificFeature(contribAnimFeatCount, 'Female')
            getContributorSpecificFeature(contribAnimFeatCount, "UNKNOWN SEX")
        
            # combine previous csv files
            createcombinedcsv(feature, "Male", "Female", "UnknownSex")
        
        else: # get specific feature list
            Male = getContributorSpecificFeatureList(contribAnimFeatCount, 'Male')
            Female = getContributorSpecificFeatureList(contribAnimFeatCount, 'Female')
            UnknownSex = getContributorSpecificFeatureList(contribAnimFeatCount, 'UNKNOWN SEX')
            
            #create combined csv file from all the previous lists
            createCombinedCsvFromList(feature, "Male", "Female", "UNKNOWN SEX", Male, Female, UnknownSex)
            
        # include sum column in csv file
        sumRows('Male-UNKNOWN SEX.csv')
        
        # plots
        createStackgraph("sumMale-UNKNOWN SEX.csv", "sex/text", "Male", "Female", 'UNKNOWN SEX')
        
    elif feature == "species/text":
        if boolean == True:
            getContributorSpecificFeature(contribAnimFeatCount, "giraffe_masai")
            getContributorSpecificFeature(contribAnimFeatCount, "zebra_plains")
        
            createcombinedcsv(feature,"giraffe_masai", "zebra_plains")
        
        else:
            giraffe = getContributorSpecificFeatureList(contribAnimFeatCount, "giraffe_masai")
            zebra = getContributorSpecificFeatureList(contribAnimFeatCount, "zebra_plains")
            
            createCombinedCsvFromList(feature, "giraffe_masai", "zebra_plains", giraffe, zebra)
        
        sumRows('giraffe_masaiandzebra_plains.csv')
        
        createStackgraph("sumgiraffe_masaiandzebra_plains.csv", "species/text", "giraffe_masai", "zebra_plains")
        
    elif feature == "yaw/text":
        if boolean == True:
            getContributorSpecificFeature(contribAnimFeatCount, "front")
            getContributorSpecificFeature(contribAnimFeatCount, "back")
            getContributorSpecificFeature(contribAnimFeatCount, "left")
            getContributorSpecificFeature(contribAnimFeatCount, "right")
            getContributorSpecificFeature(contribAnimFeatCount, "frontleft")
            getContributorSpecificFeature(contribAnimFeatCount, "frontright")
            getContributorSpecificFeature(contribAnimFeatCount, "backleft")
            getContributorSpecificFeature(contribAnimFeatCount, "backright")
        
            createcombinedcsv(feature, "front", "back", "left", "right", "frontleft", "frontright", "backleft", "backright")
        
        else:
            front = getContributorSpecificFeatureList(contribAnimFeatCount, "front")
            back = getContributorSpecificFeatureList(contribAnimFeatCount, "back")
            left = getContributorSpecificFeatureList(contribAnimFeatCount, "left")
            right = getContributorSpecificFeatureList(contribAnimFeatCount, "right")
            frontleft = getContributorSpecificFeatureList(contribAnimFeatCount, "frontleft")
            frontright = getContributorSpecificFeatureList(contribAnimFeatCount, "frontright")
            backleft = getContributorSpecificFeatureList(contribAnimFeatCount, "backleft")
            backright = getContributorSpecificFeatureList(contribAnimFeatCount, "backright")
            
            createCombinedCsvFromList(feature, "front", "back", "left", "right", "frontleft", "frontright", "backleft", "backright", front, back, left, right, frontleft, frontright, backleft, backright)
        
        sumRows('yaw_texts.csv')
        
        createStackgraph("sumyaw_texts.csv", "yaw/text", "front", "back", "left", "right", "frontleft", "frontright", "backleft", "backright")
        
    elif feature == "quality/text":
        
        if boolean == True:
            getContributorSpecificFeature(contribAnimFeatCount, "excellent")
            getContributorSpecificFeature(contribAnimFeatCount, "good")
            getContributorSpecificFeature(contribAnimFeatCount, "ok")
            getContributorSpecificFeature(contribAnimFeatCount, "poor")
            getContributorSpecificFeature(contribAnimFeatCount, "junk")
        
            createcombinedcsv("quality/text", "excellent", "good", "ok", "poor", "junk")
        else:
            excellent = getContributorSpecificFeatureList(contribAnimFeatCount, "excellent")
            good = getContributorSpecificFeatureList(contribAnimFeatCount, "good")
            ok = getContributorSpecificFeatureList(contribAnimFeatCount, "ok")
            poor = getContributorSpecificFeatureList(contribAnimFeatCount, "poor")
            junk = getContributorSpecificFeatureList(contribAnimFeatCount, "junk")
            
            createCombinedCsvFromList(feature, "excellent", "good", "ok", "poor", "junk", excellent, good, ok, poor, junk)
            
        sumRows('excellent-junk.csv')
        
        createStackgraph("sumexcellent-junk.csv", "quality/text", "excellent", "good", "ok", "poor", "junk")
    
    # have to do something new for image contributor tag   
    elif feature == 'image/contributor/tag':    #ONLY NUMBERS SO NO NEED TO CALL HELPER FUNCTION, NOT SORTED

        csv_out = csv.writer(open('contribImage_Contributor_TagMap.csv', 'w')) 
        csv_out.writerow(['CONTRIBUTOR', "image_contributor_tag"])
        for contrib, contrib2 in contribAnimFeatCount.items(): #DICTIONARY WITHIN A DICTIONARY
            for contrib3, total in contrib2.items():
                csv_out.writerow([contrib3, total])
                
        creategraph('contribImage_Contributor_TagMap.csv', feature)
    
    else:
        print("WRong feature, SOMETHING IS WRONG")
        

# Arguments : ContributorToFeatureDict , Required Specific Feature
# Accepted Specific Features: 
    # sex/text = "Male", "Female", "UNKNOWN SEX".
    # species/text = "giraffe_masai", "zebra_plains"
    # exemplar = "zero", "one"
    # yaw/text = "front", "back", "left", "right", "frontleft", "frontright", "backleft", "backright"
    # quality/text = "excellent", "good", "ok", "poor", "junk" 
# Creates: csv file 
def getContributorSpecificFeature(contribAnimFeatCount, specificfeat):
    contribSpecFeatureMap={}
    
    for contrib, feature in contribAnimFeatCount.items():
        contribSpecFeatureMap[contrib]=feature.get(specificfeat , 0)
        
    
    sortedcontribSpecFeatureMap=sorted(contribSpecFeatureMap.items(),key = operator.itemgetter(0))
    
    csv_out = csv.writer(open('contrib'+ specificfeat +'Map.csv', 'w')) 
    csv_out.writerow(['CONTRIBUTOR', specificfeat])
    for row in sortedcontribSpecFeatureMap:
        csv_out.writerow(row)


# Arguments : ContributorToFeatureDict , Required Specific Feature
# Accepted Specific Features: 
    # sex/text = "Male", "Female", "UNKNOWN SEX".
    # species/text = "giraffe_masai", "zebra_plains"
    # exemplar = "zero", "one"
    # yaw/text = "front", "back", "left", "right", "frontleft", "frontright", "backleft", "backright"
    # quality/text = "excellent", "good", "ok", "poor", "junk" 
# Return: sorted list
def getContributorSpecificFeatureList(contribAnimFeatCount, specificfeat):
    contribSpecFeatureMap={}
    
    for contrib, feature in contribAnimFeatCount.items():
        contribSpecFeatureMap[contrib]=feature.get(specificfeat , 0)
        
    
    sortedcontribSpecFeatureMap=sorted(contribSpecFeatureMap.items(),key = operator.itemgetter(0))
    
    return sortedcontribSpecFeatureMap

# Arguments : Feature name first the specific features in a list
# Returns : None
# Creates: csv file
def createcombinedcsv(*args):
    with open('contrib' + args[1] + 'Map.csv') as f:
        next(f)
        data=[tuple(line) for line in csv.reader(f)]
    
    with open('contrib' + args[2] + 'Map.csv') as f:
        next(f)
        data1=[tuple(line) for line in csv.reader(f)]
        
    if args[0] == "species/text" or args[0] == "exemplar":
        contribFeat1Df = pd.DataFrame(data,columns = ['CONTRIBUTOR',args[1]]) 
        contribFeat2Df = pd.DataFrame(data1,columns = ['CONTRIBUTOR',args[2]]) 
    
        Feat1Feat2Merge = pd.merge(contribFeat1Df,contribFeat2Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
    
        Feat1Feat2Merge.to_csv(args[1] +"and" +args[2] 'and' + args[2] + '.csv',index=False)
    elif args[0] == "sex/text":
        with open('contrib' + args[3] + 'Map.csv') as f:
            next(f)
            data2=[tuple(line) for line in csv.reader(f)]
    
        contribFeat1Df = pd.DataFrame(data,columns = ['CONTRIBUTOR', args[1]]) 
        contribFeat2Df = pd.DataFrame(data1,columns = ['CONTRIBUTOR', args[2]]) 
        contribFeat3Df = pd.DataFrame(data2,columns = ['CONTRIBUTOR', args[3]])

        Feat1Feat2Merge = pd.merge(contribFeat1Df,contribFeat2Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        FinalContribDf = pd.merge(Feat1Feat2Merge,contribFeat3Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')

        FinalContribDf.to_csv(args[1]+ '-'+ args[3] + '.csv',index=False)
        
    elif args[0] == "quality/text" or args[0] == "age/months":
        with open('contrib' + args[3] + 'Map.csv') as f:
            next(f)
            data2=[tuple(line) for line in csv.reader(f)]
        
        with open('contrib' + args[4] + 'Map.csv') as f:
            next(f)
            data3=[tuple(line) for line in csv.reader(f)]
        
        with open('contrib' + args[5] + 'Map.csv') as f:
            next(f)
            data4=[tuple(line) for line in csv.reader(f)]
     
        contribFeat1Df = pd.DataFrame(data,columns = ['CONTRIBUTOR', args[1]]) 
        contribFeat2Df = pd.DataFrame(data1,columns = ['CONTRIBUTOR', args[2]]) 
        contribFeat3Df = pd.DataFrame(data2,columns = ['CONTRIBUTOR',args[3]])
        contribFeat4Df = pd.DataFrame(data3,columns = ['CONTRIBUTOR',args[4]])
        contribFeat5Df = pd.DataFrame(data4,columns = ['CONTRIBUTOR',args[5]])
    
        Feat1Feat2Merge = pd.merge(contribFeat1Df,contribFeat2Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        Feat2Feat3Merge = pd.merge(Feat1Feat2Merge,contribFeat3Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        Feat3Feat4Merge = pd.merge(Feat2Feat3Merge,contribFeat4Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        FinalContribDf = pd.merge(Feat3Feat4Merge,contribFeat5Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
    
        FinalContribDf.to_csv(args[1] '-' + args[5] + '.csv',index=False)
        
    elif args[0] == "yaw/text":
        with open('contrib' + args[3] + 'Map.csv') as f:
            next(f)
            data2=[tuple(line) for line in csv.reader(f)]
        
        with open('contrib' + args[4] + 'Map.csv') as f:
            next(f)
            data3=[tuple(line) for line in csv.reader(f)]
        
        with open('contrib' + args[5] + 'Map.csv') as f:
            next(f)
            data4=[tuple(line) for line in csv.reader(f)]
        
        with open('contrib' + args[6] + 'Map.csv') as f:
            next(f)
            data5=[tuple(line) for line in csv.reader(f)]
        
        with open('contrib' + args[7] + 'Map.csv') as f:
            next(f)
            data6=[tuple(line) for line in csv.reader(f)]
        
        with open('contrib' + args[8] + 'Map.csv') as f:
            next(f)
            data7=[tuple(line) for line in csv.reader(f)]
        
        contribFeat1Df = pd.DataFrame(data,columns = ['CONTRIBUTOR', args[1]]) 
        contribFeat2Df = pd.DataFrame(data1,columns = ['CONTRIBUTOR', args[2]]) 
        contribFeat3Df = pd.DataFrame(data2,columns = ['CONTRIBUTOR',args[3]])
        contribFeat4Df = pd.DataFrame(data3,columns = ['CONTRIBUTOR',args[4]])
        contribFeat5Df = pd.DataFrame(data4,columns = ['CONTRIBUTOR',args[5]])
        contribFeat6Df = pd.DataFrame(data5,columns = ['CONTRIBUTOR',args[6]])
        contribFeat7Df = pd.DataFrame(data6,columns = ['CONTRIBUTOR',args[7]])
        contribFeat8Df = pd.DataFrame(data7,columns = ['CONTRIBUTOR',args[8]])

        Feat1Feat2Merge = pd.merge(contribFeat1Df,contribFeat2Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        Feat2Feat3Merge = pd.merge(Feat1Feat2Merge,contribFeat3Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        Feat3Feat4Merge = pd.merge(Feat2Feat3Merge,contribFeat4Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        Feat4Feat5Merge = pd.merge(Feat3Feat4Merge,contribFeat5Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        Feat5Feat6Merge = pd.merge(Feat4Feat5Merge,contribFeat6Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        Feat6Feat7Merge = pd.merge(Feat5Feat6Merge,contribFeat7Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        FinalContribDf = pd.merge(Feat6Feat7Merge,contribFeat8Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')


        FinalContribDf.to_csv(args[0]+'yaw_texts.csv',index=False) 


# Arguments : feature, specific feature and lists
# Accepted Specific Features: sex_texts = "Male", "Female", "UNKNOWN SEX", etc.
# Creates: combined csv files from lists
def createCombinedCsvFromList(*args): 
    if args[0] == "species/text" or args[0] == "exemplar":
        data=[tuple(line) for line in args[3]]
        data1=[tuple(line) for line in args[4]]
        
        contribFeat1Df = pd.DataFrame(data,columns = ['CONTRIBUTOR',args[1]]) 
        contribFeat2Df = pd.DataFrame(data1,columns = ['CONTRIBUTOR',args[2]]) 
    
        Feat1Feat2Merge = pd.merge(contribFeat1Df,contribFeat2Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
    
        Feat1Feat2Merge.to_csv(args[1] + 'and' + args[2] + '.csv',index=False) 

    elif args[0] == "sex/text":
        data=[tuple(line) for line in args[4]]
        data1=[tuple(line) for line in args[5]]
        data2=[tuple(line) for line in args[6]]
    
        contribFeat1Df = pd.DataFrame(data,columns = ['CONTRIBUTOR', args[1]]) 
        contribFeat2Df = pd.DataFrame(data1,columns = ['CONTRIBUTOR', args[2]]) 
        contribFeat3Df = pd.DataFrame(data2,columns = ['CONTRIBUTOR', args[3]])

        Feat1Feat2Merge = pd.merge(contribFeat1Df,contribFeat2Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        FinalContribDf = pd.merge(Feat1Feat2Merge,contribFeat3Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')

        FinalContribDf.to_csv(args[1] + "-"+ args[3] + '.csv',index=False)
        
    elif args[0] == "quality/text" or args[0] == "age/months":
        data=[tuple(line) for line in args[6]]
        data1=[tuple(line) for line in args[7]]
        data2=[tuple(line) for line in args[8]]
        data3=[tuple(line) for line in args[9]]
        data4=[tuple(line) for line in args[10]]
        
        contribFeat1Df = pd.DataFrame(data,columns = ['CONTRIBUTOR', args[1]]) 
        contribFeat2Df = pd.DataFrame(data1,columns = ['CONTRIBUTOR', args[2]]) 
        contribFeat3Df = pd.DataFrame(data2,columns = ['CONTRIBUTOR',args[3]])
        contribFeat4Df = pd.DataFrame(data3,columns = ['CONTRIBUTOR',args[4]])
        contribFeat5Df = pd.DataFrame(data4,columns = ['CONTRIBUTOR',args[5]])
    
        Feat1Feat2Merge = pd.merge(contribFeat1Df,contribFeat2Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        Feat2Feat3Merge = pd.merge(Feat1Feat2Merge,contribFeat3Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        Feat3Feat4Merge = pd.merge(Feat2Feat3Merge,contribFeat4Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        FinalContribDf = pd.merge(Feat3Feat4Merge,contribFeat5Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
    
        FinalContribDf.to_csv(args[1] + '-' + args[5] + '.csv',index=False)
        
    elif args[0] == "yaw/text":
        data=[tuple(line) for line in args[9]]
        data1=[tuple(line) for line in args[10]]
        data2=[tuple(line) for line in args[11]]
        data3=[tuple(line) for line in args[12]]
        data4=[tuple(line) for line in args[13]]
        data5=[tuple(line) for line in args[14]]
        data6=[tuple(line) for line in args[15]]
        data7=[tuple(line) for line in args[16]]
        
        contribFeat1Df = pd.DataFrame(data,columns = ['CONTRIBUTOR', args[1]]) 
        contribFeat2Df = pd.DataFrame(data1,columns = ['CONTRIBUTOR', args[2]]) 
        contribFeat3Df = pd.DataFrame(data2,columns = ['CONTRIBUTOR',args[3]])
        contribFeat4Df = pd.DataFrame(data3,columns = ['CONTRIBUTOR',args[4]])
        contribFeat5Df = pd.DataFrame(data4,columns = ['CONTRIBUTOR',args[5]])
        contribFeat6Df = pd.DataFrame(data5,columns = ['CONTRIBUTOR',args[6]])
        contribFeat7Df = pd.DataFrame(data6,columns = ['CONTRIBUTOR',args[7]])
        contribFeat8Df = pd.DataFrame(data7,columns = ['CONTRIBUTOR',args[8]])

        Feat1Feat2Merge = pd.merge(contribFeat1Df,contribFeat2Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        Feat2Feat3Merge = pd.merge(Feat1Feat2Merge,contribFeat3Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        Feat3Feat4Merge = pd.merge(Feat2Feat3Merge,contribFeat4Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        Feat4Feat5Merge = pd.merge(Feat3Feat4Merge,contribFeat5Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        Feat5Feat6Merge = pd.merge(Feat4Feat5Merge,contribFeat6Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        Feat6Feat7Merge = pd.merge(Feat5Feat6Merge,contribFeat7Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')
        FinalContribDf = pd.merge(Feat6Feat7Merge,contribFeat8Df,left_on = 'CONTRIBUTOR',right_on = 'CONTRIBUTOR')


        FinalContribDf.to_csv('yaw_texts.csv',index=False) 


# Arguments : CSV file
# Returns : None
# Creates: csv file with sum column
def sumRows(filename, header=False):  
    with open(filename,'r') as csvfile:
        with open('sum' + filename, 'w') as csvoutput:
            writer = csv.writer(csvoutput, lineterminator='\n')
            reader = csv.reader(csvfile)

            all = []
            row = next(reader)
            row.append('Sum')
            all.append(row)

            for row in reader:
                rowtotal = 0
                for column in row[1:]:
                    rowtotal += int(column)
                row.append(rowtotal)
                all.append(row)

            writer.writerows(all)


# Arguments: Filename, feature, and required specific features in a list in that order
# Creates: a stacked bar graph
def createStackgraph(*args):
    
    if args[1] == "species/text" or args[1] == "exemplar":
        data=pd.read_csv(args[0],  usecols=(0,1,2,3))
    elif args[1] == "sex/text":
        data=pd.read_csv(args[0],  usecols=(0,1,2,3,4))
    elif args[1] == "quality/text" or args[1] == "age/months":
        data=pd.read_csv(args[0],  usecols=(0,1,2,3,4,5,6))
    elif args[1] == "yaw/text":
        data=pd.read_csv(args[0],  usecols=(0,1,2,3,4,5,6,7,8,9))
    else:
        print("Error")
        
    data = data.sort_values('Sum', ascending=False)
    width = 0.50
    ind = np.arange(56) + 0.75
    
    
    if args[1] == "species/text" or args[1] == "exemplar":
        p1=plt.bar(ind, data.loc[:, args[2]], width, color = 'b') 
        p2=plt.bar(ind, data.loc[:, args[3]], width, color = 'r', bottom=data.loc[:, args[2]])

        plt.legend((p1[0], p2[0]), (args[2], args[3]))
        
    elif args[1] == "sex/text":
        p1=plt.bar(ind, data.loc[:, args[2]], width, color = 'b')
        p2=plt.bar(ind, data.loc[:, args[3]], width, color = 'r', bottom=data.loc[:, args[2]])
        p3=plt.bar(ind, data.loc[:, args[4]], width, color = 'g', bottom=data.loc[:, args[2]] +data.loc[:, args[3]])

        plt.legend((p1[0], p2[0], p3[0]), (args[2], args[3], args[4]))
        
    elif args[1] == "quality/text" or args[1] == "age/months":
        p1=plt.bar(ind, data.loc[:, args[2]], width, color = 'b')
        p2=plt.bar(ind, data.loc[:, args[3]], width, color = 'r', bottom=data.loc[:, args[2]])
        p3=plt.bar(ind, data.loc[:, args[4]], width, color = 'g', bottom=data.loc[:, args[2]] +data.loc[:, args[3]])
        p4=plt.bar(ind, data.loc[:, args[5]], width, color = 'y', bottom=data.loc[:, args[2]] +data.loc[:, args[3]] +data.loc[:, args[4]])
        p5=plt.bar(ind, data.loc[:, args[6]], width, color = 'k', bottom=data.loc[:, args[2]] +data.loc[:, args[3]] +data.loc[:, args[4]] +data.loc[:, args[5]])
        
        plt.legend((p1[0], p2[0], p3[0], p4[0], p5[0]), (args[2], args[3], args[4], args[5], args[6]))
        
    elif args[1] == "yaw/text":
        p1=plt.bar(ind, data.loc[:, args[2]], width, color = 'b')
        p2=plt.bar(ind, data.loc[:, args[3]], width, color = 'r', bottom=data.loc[:, args[2]])
        p3=plt.bar(ind, data.loc[:, args[4]], width, color = 'g', bottom=data.loc[:, args[2]] +data.loc[:, args[3]])
        p4=plt.bar(ind, data.loc[:, args[5]], width, color = 'y', bottom=data.loc[:, args[2]] +data.loc[:, args[3]] +data.loc[:, args[4]])
        p5=plt.bar(ind, data.loc[:, args[6]], width, color = 'k', bottom=data.loc[:, args[2]] +data.loc[:, args[3]] +data.loc[:, args[4]] +data.loc[:, args[5]])
        p6=plt.bar(ind, data.loc[:, args[7]], width, color = 'c', bottom=data.loc[:, args[2]] +data.loc[:, args[3]] +data.loc[:, args[4]] +data.loc[:, args[5]] + data.loc[:, args[6]])
        p7=plt.bar(ind, data.loc[:, args[8]], width, color = 'm', bottom=data.loc[:, args[2]] +data.loc[:, args[3]] +data.loc[:, args[4]] +data.loc[:, args[5]] + data.loc[:, args[6]]+ data.loc[:, args[7]])
        p8=plt.bar(ind, data.loc[:, args[9]], width,color = 'tan', bottom=data.loc[:, args[2]] +data.loc[:, args[3]] +data.loc[:, args[4]] +data.loc[:, args[5]] + data.loc[:, args[6]]+ data.loc[:, args[7]] +data.loc[:, args[8]])

        plt.legend((p1[0], p2[0], p3[0], p4[0], p5[0], p6[0], p7[0], p8[0]), (args[2], args[3], args[4], args[5], args[6], args[7], args[8], args[9]))
    else:
        print("error")
        
    plt.xticks(ind, data.loc[:, 'CONTRIBUTOR'], rotation=90)
    plt.rcParams['xtick.labelsize'] = 8
    plt.ylabel('Total Number of Animals')
    plt.xlabel('Contributor')

    #plt.show() #get rid of to save image
    
    plt.savefig(str("../"+args[2] +"_expt2.png"),bbox_inches='tight') # can't put feature because it says can't find directory ex. sex/text


# Arguments : csv_file , Required Specific Feature
# Accepted Specific Features: sex_texts = "Male", "Female", "UNKNOWN SEX", etc.
# Creates: plot
def creategraph(csv_file, specific_feature):
    data = pd.read_csv(csv_file, sep=',',header=0, index_col =0) #csv_file

    data = data.sort_values('image_contributor_tag', ascending=False)
    
    data.plot(kind='bar')
    plt.rcParams['xtick.labelsize'] = 5
    plt.ylabel('Number of ' + specific_feature + ' taken')
    plt.xlabel('Contributor')
    plt.title('Contributor to '+ specific_feature + ' Totals')

    #plt.show()
    
    specific_feature = 'image_contributor_tag'
    
    plt.savefig(str("../"+specific_feature+"_expt2.png") ,bbox_inches='tight')
 

