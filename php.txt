#! /usr/bin/env python3
import webbrowser, requests, threading, os, sys, platform, time, re

"""Script for downloading any file from an URL."""

#download-progression
def progression():
    clear()
    completion=0
    delimiter=100
    while completion<delimiter:
        #resets completion 
        completion=0
        print(f"Downloading all links in file {links_file}...\n")
        for k,v in tread_dl_prog.items():
            sys.stdout.write(f"{bcolors.BOLD}{v}%{bcolors.ENDC} Completed of {k}...\n")
            completion+=int(v)/len(tread_dl_prog)
        clear()

def seq_progression():
    clear()
    while ongoing:
        print(f"Downloading all links in file {links_file}...\n")
        for k,v in tread_dl_prog.items():
            sys.stdout.write(f"{bcolors.BOLD}{v}%{bcolors.ENDC} Completed of {k}...\n")
        clear()

#function for each video download Thread
def download(video_link,iteration,class_name):
    #handle exception for clean console; download would still run otherwise
    try:
        r=requests.get(video_link,stream=True)
    except requests.exceptions.MissingSchema:
        pass
    else:
        type_=r.headers.get('content-type')
        match=0
        if type_ is None:
            match=link[-4:]
        else:
            regex=re.compile(r'/\w\w\w|/\w\w\w')
            match=regex.search(type_)
            match=match.group()
            match="."+match[1:]
        #Without class_name
        if class_name=="":
            #establish destination's file path and name
            recipient=os.path.join(downloads_,'_Video_'+str(iteration+1)+match)
        #With class_name
        else:
            #establish folder's path and name
            folder=os.path.join(downloads_,class_name)
            #avoid overwriting existing folder from an eventual preexisting thread having created it already
            try:
                os.mkdir(folder)
            except FileExistsError:
                pass
            #establish destination's file path and name
            recipient=os.path.join(folder,'_Video_'+str(iteration+1)+match)
        #write file to destination
        with open(recipient, 'wb') as f:
            total_length = r.headers.get('content-length')
            #no content length header; writes file all the same
            if total_length is None:
                f.write(r.content)
            #write file, meanwhile populating dictionary tread_dl_prog for progression())
            else:
                dl = 0
                total_length = int(total_length)
                for data in r.iter_content(chunk_size=100000):
                    dl += len(data)
                    f.write(data)
                    done = int(dl *100/ total_length)
                    tread_dl_prog[recipient]=str(done)

#declares link iterator for composition of the destination's file name
i=0
#declares list of downloadind threads
downloadThreads=[]
#declares dictionary for threads' download progressions
tread_dl_prog={}
#clear console function
clear=lambda:os.system(sysclear)
if platform.system()=='Darwin':
    desktop = os.path.join(os.path.join(os.environ['PWD']), 'Desktop')
    downloads_= os.path.join(os.path.join(os.environ['PWD']), 'Downloads')
    sysclear='clear'
elif platform.system()=='Windows':
    desktop = os.path.join(os.path.join(os.environ['USERPROFILE']), 'Desktop')
    downloads_= os.path.join(os.path.join(os.environ['USERPROFILE']), 'Downloads')
    sysclear='cls'
else:
    print('platform error101')
    time.sleep(5)
    sys.exit(0)
class bcolors:
    HEADER = '\033[95m'
    OKBLUE = '\033[94m'
    OKGREEN = '\033[92m'
    ENDC = '\033[0m'
    BOLD = '\033[1m'

#copyright
clear()
print(bcolors.HEADER+'©2020TETPSIIEFPPTM\n'+bcolors.ENDC)
#greeting
while True:
    try:
        links_file=input("What is the name of the .txt file on your Desktop folder containing the links?\n")
        #gets user's desktop and downloads folder path depending on OS(Linux missing)
        #defines links.txt source
        links_file= os.path.join(desktop,links_file+'.txt')
        #open .txt links file and create list of links
        with open(links_file) as source_list:
            contents=source_list.read()
            links=contents.split("\n")
        break
    except FileNotFoundError:
        print("File does not exist...")

#Choose download option
while True:
    print("\nChoose your download option:")
    dl_option=input("1-Execute all links at once(Recommended only if you have the bandwith)\n"
                    "2-Execute each link at a time.\n")
    if dl_option=="1" or dl_option=="2":
        break
    else:
        print("Not an option...")
        continue

print('\nBeginning files\' download...')
#download links all at a time
if dl_option=="1":
    then=time.time()
    for link in links:
        link.strip()#in case the stupid user leaves blankspace 
        if link!="":#skip blanklines
            if link[0]=='(' and link[-1]==')':
                collec_name=link[1:-1]
                i=0
            else:
                #initiate downloading thread for each link
                try:#NameError exception handling(collec_name is not defined meaning User did not provide folder name)
                    downloadThread=threading.Thread(target=download,args=(link,i,collec_name))
                except NameError:
                    collec_name=""
                    downloadThread=threading.Thread(target=download,args=(link,i,collec_name))
                downloadThreads.append(downloadThread)
                downloadThread.start()
                i+=1
    #wait for all threads to end
    progThread=threading.Thread(target=progression)
    progThread.start()
    progThread.join()
    for downloadThread in downloadThreads:
        downloadThread.join()
#each link at a time
elif dl_option=="2":
    then=time.time()
    ongoing=True
    progThread=threading.Thread(target=seq_progression)
    progThread.start()
    for link in links:
        link.strip()#in case the stupid user leaves blankspace 
        if link!="":#skip blanklines
            if link[0]=='(' and link[-1]==')':
                collec_name=link[1:-1]
                i=0
            #initiate download for each link at a time
            else:
                #NameError exception handling(collec_name is not defined meaning User did not provide folder name)
                try:
                    downloadThread=threading.Thread(target=download,args=(link,i,collec_name))
                except NameError:
                    collec_name=""
                    downloadThread=threading.Thread(target=download,args=(link,i,collec_name))
                downloadThread.start()
                downloadThread.join()
                i+=1
    ongoing=False
    progThread.join()

#allow for 100% completion to be shown on all files after progression() has ended, otherwise one would always be left at 99% or lower
print(f"{bcolors.OKBLUE}You're welcome{bcolors.ENDC}\n")
for k,v in tread_dl_prog.items():
    sys.stdout.write(f"{v}% Completed of {k}...\n")

now=time.time()
#confirm all downloads have finished
print(bcolors.OKGREEN+'\nAll files have been Downloaded and are stored in your Downloads folder.'+bcolors.ENDC)
print(f"{bcolors.OKBLUE}Process finished in {bcolors.ENDC}{bcolors.OKGREEN}{round(now-then)}{bcolors.ENDC}{bcolors.OKBLUE} seconds. ;-){bcolors.ENDC}\n")
for i in range(0,11):
    countdown=10-i
    sys.stdout.write(f"\rGoodbye in {countdown}... ")
    sys.stdout.flush()
    if countdown!=0:
        time.sleep(1)
sys.stdout.write(f"{bcolors.BOLD}\rGoodbye.{bcolors.ENDC}")
sys.stdout.flush()
sys.exit(0)