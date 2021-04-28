
import xml.etree.ElementTree as ET
import spacy
import en_core_web_sm
import pathlib
import os
from graphviz import Digraph
from collections import Counter
import matplotlib.pyplot as plt

nlp = en_core_web_sm.load()
nlp = spacy.load("en_core_web_sm")

all_data = []
path_file = pathlib.Path().absolute()

#read xml datei
for subdir, dirs, files in os.walk(path_file):               #skip hidden dirs and files
    # skip all hidden directories and files
    files = [f for f in files if not f[0] == '.']
    dirs[:] = [d for d in dirs if not d[0] == '.' and not d[0:4] == 'venv']
    if not subdir.startswith('.'):
        for f_name in files:
            path = subdir + os.sep + f_name
            if path.endswith(".xml"):
                #parse xml files
                tree = ET.parse(path)
                root = tree.getroot()
                all_data.append(root)
                if path.endswith("Bicycles.xml"):
                    tree_b = ET.parse(path)
                    root_b = tree_b.getroot()
                if path.endswith("Highlights_of_the_Prado_Museum.xml"):
                    tree_hotpm = ET.parse(path)
                    root_hotpm = tree_hotpm.getroot()

def count_stuff(cs, ts):                        #count the number of wanted
    count_set = set(cs)
    count_temp = ts
    count_dict = {}

    for count in count_set:
        counter = 0
        for typ in count_temp:
            if count == typ:
                counter += 1
                count_dict[count] = counter
    return count_dict


#counts the PoS
x = 0
while x < len(all_data):
    print(Counter(([token.pos_ for token in nlp(all_data[x][0].text)])))
    x += 1





#counts place, etc.
spatial_entities = 0
place = 0
motions = 0
locations = 0
signals = 0
QsLinks = 0
OLinks = 0

for mark in all_data:
    for tags in mark[1]:
        if tags.tag == "SPATIAL_ENTITY":
            spatial_entities += 1
        elif tags.tag == "PLACE":
            place += 1
        elif tags.tag == "MOTION":
            motions += 1
        elif tags.tag == "LOCATION":
            locations += 1
        elif tags.tag == "MOTION_SIGNAL":
            signals += 1
        elif tags.tag == "QSLINK":
            QsLinks += 1
        elif tags.tag == "OLINK":
            OLinks += 1
        else:
            pass

print("Spatial_entities:", spatial_entities)
print("Place:", place)
print("Motions:", motions)
print("Locations:", locations)
print("Signals:", signals)
print("QsLinks:", QsLinks)
print("OLinks:", OLinks)



#counts qslink types
types_counter = set()
temp = []
QsLink_typ = []

for mark in all_data:
    for qslink in mark[1]:
        if qslink.tag == "QSLINK":
            types_counter.add(qslink.get('relType'))
            temp.append(qslink.get('relType'))
            QsLink_typ = count_stuff(types_counter, temp)
        else:
            pass

print(QsLink_typ.items())

#sentence length
to_point = []
hm = 0
sen_len = set()
plot_data = {}

x = 0
while x < len(all_data):
    doc = nlp(all_data[x][0].text)
    x += 1
    for sents in doc.sents:
        hm = len(sents)
        to_point.append(hm)
sen_len = set(to_point)
plot_data = count_stuff(sen_len, to_point)

#grafic
names = list(plot_data.keys())
values = list(plot_data.values())
plt.bar(names, values)
plt.xlabel("Satzlaenge")
plt.title('Average sentence length')
plt.show()


qsl = []
ol = []
qsl_text = []
ol_text = []
qsl_set = set()
ol_set = set()

#search the triggers and they text
for mark in all_data:
    trig_dict = {}
    one_way_qsl = []
    one_way_ol = []
    for praep in mark[1]:
        if praep.tag == "SPATIAL_SIGNAL":
            trig_dict[praep.get('id')] = praep.get('text')
        if praep.tag == "QSLINK":
            one_way_qsl.append(praep.get('trigger'))
        elif praep.tag == "OLINK":
            one_way_ol.append(praep.get('trigger'))

    for change in one_way_qsl:                                         #exchange the id with the text
        if change != '':
            try:
                qsl_set.add(trig_dict[change])
                qsl_text.append(trig_dict[change])
                qsl_count = count_stuff(qsl_set, qsl_text)
            except: pass

    for change in one_way_ol:
        if change != '':
            try:
                ol_set.add(trig_dict[change])
                ol_text.append(trig_dict[change])
                ol_count = count_stuff(ol_set, ol_text)
            except: pass

print("QSLinks:" , qsl_count, "OLinks:", ol_count)

#count the motion verbs
mot_verb = []
mw_set = set()
for mark in all_data:
    for motion in mark[1]:
        if motion.tag == "MOTION":
            mot_verb.append(motion.get('text'))
            mw_set.add(motion.get('text'))
            most_mw = count_stuff(mw_set, mot_verb)
            
s_most_mw= sorted(most_mw.items(), key=lambda item: item[1])            #only the five biggest one
print(s_most_mw[:-6:-1])                                                #cause from small to large take the last five things


n_place = []
n_location = []
n_spatial_entity = []
n_non_motion_event = []
n_path = []

for node in root_b[1]:
    if node.tag == "PLACE":
        n_place.append([node.get('id'), node.get('text')])
    elif node.tag == "LOCATION":
        n_location.append((node.get('id'), node.get('text')))
    elif node.tag == "SPATIAL_ENTITY":
        n_spatial_entity.append([node.get('id'), node.get('text')])
    elif node.tag == "NONMOTION_EVENT":
        n_non_motion_event.append([node.get('id'), node.get('text')])
    elif node.tag == "PATH":
        n_path.append([node.get('id'), node.get('text')])

dot = Digraph('dot', filename='bicycle.gv', engine='sfdp')

for dot_node in n_place:
    if dot_node[1] != '':
        dot.node(dot_node[1], color='red')
    else:
        dot.node(dot_node[0], color='red')
for dot_node in n_location:
    if dot_node[1] != '':
        dot.node(dot_node[1], color='orange')
    else:
        dot.node(dot_node[0], color='orange')
for dot_node in n_spatial_entity:
    if dot_node[1] != '':
        dot.node(dot_node[1], color='yellow')
    else:
        dot.node(dot_node[0], color='yellow')
for dot_node in n_non_motion_event:
    if dot_node[1] != '':
        dot.node(dot_node[1], color='green')
    else:
        dot.node(dot_node[0], color='green')
for dot_node in n_path:
    if dot_node[1] != '':
        dot.node(dot_node[1], color='blue')
    else:
        dot.node(dot_node[0], color='blue')


place_n = []
location_n = []
spatial_entity_n = []
nme_n = []
path_n = []

for node in root_hotpm[1]:
    if node.tag == "PLACE":
        place_n.append([node.get('id'), node.get('text')])
    elif node.tag == "LOCATION":
        location_n.append((node.get('id'), node.get('text')))
    elif node.tag == "SPATIAL_ENTITY":
        spatial_entity_n.append([node.get('id'), node.get('text')])
    elif node.tag == "NONMOTION_EVENT":
        nme_n.append([node.get('id'), node.get('text')])
    elif node.tag == "PATH":
        path_n.append([node.get('id'), node.get('text')])

dot = Digraph('dot', filename='Highlights_of_the_Prado_Museum.xml.gv', engine='sfdp')

for dot_node in n_place:
    if dot_node[1] != '':
        dot.node(dot_node[1], color='red')
    else:
        dot.node(dot_node[0], color='red')
for dot_node in n_location:
    if dot_node[1] != '':
        dot.node(dot_node[1], color='orange')
    else:
        dot.node(dot_node[0], color='orange')
for dot_node in n_spatial_entity:
    if dot_node[1] != '':
        dot.node(dot_node[1], color='yellow')
    else:
        dot.node(dot_node[0], color='yellow')
for dot_node in nme_n:
    if dot_node[1] != '':
        dot.node(dot_node[1], color='green')
    else:
        dot.node(dot_node[0], color='green')
for dot_node in n_path:
    if dot_node[1] != '':
        dot.node(dot_node[1], color='blue')
    else:
        dot.node(dot_node[0], color='blue')



# Press the green button in the gutter to run the script.
# if __name__ == '__main__':
