# -from urllib.request import urlopen
from urllib.error import HTTPError
from bs4 import BeautifulSoup
import pymysql
import os
import time
import re
def analysis(surl):
    genres=''
    name=''
    director=''
    actors=''
    wri=''
    area=''
    nameList=''
    lang=''
    runtime=''
    date=''
    intro=''
    pic=''
    sco=''
    try:
        html = urlopen(surl)
    except HTTPError as e:
        print(e)
        
    else:
        bsObj = BeautifulSoup(html)
        try:
            if(bsObj.find('span',{'property':'v:itemreviewed'})):
                name     = bsObj.find('span',{'property':'v:itemreviewed'}).get_text()
            else:
                return 0
            if(bsObj.find("a", {"rel":"v:directedBy"})):
                
                director = bsObj.find("a", {"rel":"v:directedBy"}).get_text()
            if(bsObj.find(text='编剧')!=None):
                if(str(type(bsObj.find(text='编剧').next_element.next_element))=='<class \'bs4.element.Tag\'>'):
                    wri  = bsObj.find(text='编剧').next_element.next_element.get_text()
                    if(len(wri)>255):
                        wri=wri[:255]
                        wri=wri.rstrip(' abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ')
            if(bsObj.find('span',{'class':"actor"})):
                
                actors   = bsObj.find('span',{'class':"actor"}).get_text()
                actors=actors.lstrip('主演: ')
                if(len(actors)>255):
                    actors=actors[:255]
                    actors=actors.rstrip(' abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ')
            if(bsObj.find(text='制片国家/地区:')):
                area     = bsObj.find(text='制片国家/地区:').next_element
            nameList = bsObj.findAll("span", {"property":"v:genre"})
            if(nameList!=''):
                for genre in nameList:
                    genres=genres+genre.get_text()+'/'
                genres=genres.rstrip('/')
            if(bsObj.find(text='语言:')):
                lang     = bsObj.find(text='语言:').next_element
            if(bsObj.find('span',{'property':"v:initialReleaseDate"})):
                date     = bsObj.find('span',{'property':"v:initialReleaseDate"}).get_text()
            if(bsObj.find('span',{'property':"v:runtime"})):
                runtime  = bsObj.find('span',{'property':"v:runtime"}).get_text()
            if(bsObj.find('span',{'class':"all hidden"})!=None):
                intro    = bsObj.find('span',{'class':"all hidden"}).get_text()
                intro=intro.strip()
                intro=intro[:5000]
            else:
                if(bsObj.find('span',{'property':'v:summary'})):
                    intro    =bsObj.find('span',{'property':'v:summary'}).get_text()
                    intro=intro.strip()
            if(bsObj.find('div',{'id':'mainpic'}).a.img['src']!=None):
                pic      = bsObj.find('div',{'id':'mainpic'}).a.img['src']
            if(bsObj.find('strong',{'property':"v:average"})):
                sco      = bsObj.find('strong',{'property':"v:average"}).get_text()
                print('2')
        except AttributeError as e:
            print(e)
            conn=pymysql.connect(host='localhost',port=3306,user='root',passwd='password',db='mysql',charset='utf8')
            cursor=conn.cursor()
            if(sco!=''):
                cursor.execute("insert into film(FILM_NAME,FILM_DIREC,FILM_WRI,FILM_HERO,FILM_AREA,FILM_TYPE,FILM_LANG,FILM_DATE,FILM_LEN,FILM_INTRO,FILM_IMG,FILM_DBSC)values(%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s)",(name,director,wri,actors,area,genres,lang,date,runtime,intro,pic,sco))
            else:
                cursor.execute("insert into film(FILM_NAME,FILM_DIREC,FILM_WRI,FILM_HERO,FILM_AREA,FILM_TYPE,FILM_LANG,FILM_DATE,FILM_LEN,FILM_INTRO,FILM_IMG)values(%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s)",(name,director,wri,actors,area,genres,lang,date,runtime,intro,pic))
            conn.commit()
            cursor.close()
            conn.close()
            time.sleep(0.9)
            
        else:
            conn=pymysql.connect(host='localhost',port=3306,user='root',passwd='password',db='mysql',charset='utf8')
            cursor=conn.cursor()
            if(sco!=''):
                cursor.execute("insert into film(FILM_NAME,FILM_DIREC,FILM_WRI,FILM_HERO,FILM_AREA,FILM_TYPE,FILM_LANG,FILM_DATE,FILM_LEN,FILM_INTRO,FILM_IMG,FILM_DBSC)values(%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s)",(name,director,wri,actors,area,genres,lang,date,runtime,intro,pic,sco))
            else:
                cursor.execute("insert into film(FILM_NAME,FILM_DIREC,FILM_WRI,FILM_HERO,FILM_AREA,FILM_TYPE,FILM_LANG,FILM_DATE,FILM_LEN,FILM_INTRO,FILM_IMG)values(%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s)",(name,director,wri,actors,area,genres,lang,date,runtime,intro,pic))
            conn.commit()
            cursor.close()
            conn.close()
            #print(name)
            #print(director)
            #print(wri)
            #print(actors)
            #print(area)
            #print(genres)
            #print(sco)
            #print(date)
            #print(intro)
            #print(lang)
            #print(pic)
            time.sleep(0.9)
conn = pymysql.connect(host='localhost', port=3306, user='root', passwd='password', db='filmid', charset='utf8')
cursor = conn.cursor()
many = cursor.execute("select film_id from film")
filmid=cursor.fetchmany(many)
cursor.close()
conn.close()
for id in filmid:
    analysis('https://movie.douban.com/subject/' + str(id[0]))
    print(id[0])
    

