# python-Uninstall
Uninstall something Apk and something fill
# -*- coding: utf-8 -*-
# __author__ = 'xinjiu.qiao'
from random import choice
import re
import threading
import urllib
import os
import urllib2
import ssl
import xlrd
import socket
import time,sys
url = "https://m.apkpure.com/google-play-movies-tv/com.google.android.videos/download?from=details"
headers = {"User-Agent": "Mozilla/5.0 (Windows NT 6.1; rv:40.0) Gecko/20100101 Firefox/40.0"}
req = urllib2.Request(url, headers=headers)
sslPROTOCOL = [ssl.PROTOCOL_SSLv2, ssl.PROTOCOL_SSLv23, ssl.PROTOCOL_TLSv1, ssl.PROTOCOL_TLSv1_1,
               ssl.PROTOCOL_TLSv1_2]

def get_protocol():

    for i in sslPROTOCOL:
        try:
            f = urllib2.urlopen(req, context=ssl.SSLContext(i))
            break
        except:
            continue
    return i
user_agents = [
    'Mozilla/5.0 (Windows; U; Windows NT 5.1; it; rv:1.8.1.11) Gecko/20071127 Firefox/2.0.0.11',
    'Opera/9.25 (Windows NT 5.1; U; en)',
    'Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322; .NET CLR 2.0.50727)',
    'Mozilla/5.0 (compatible; Konqueror/3.5; Linux) KHTML/3.5.5 (like Gecko) (Kubuntu)',
    'Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.8.0.12) Gecko/20070731 Ubuntu/dapper-security Firefox/1.5.0.12',
    'Lynx/2.8.5rel.1 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/1.2.9'
]
couter_lock = threading.Lock()  # 定义多线程锁
class MyOpener(urllib.FancyURLopener, object):
    version = choice(user_agents)
class download():
    def __init__(self):
        self.sheet = u'工作任务'
        self.file = u'F:\\apk\\zwf1.xlsx'
        self.apkpath =u'F:\\apk\\zwf1'
        self.delete_apk = 'true'
    def get_values(self):
        workbook = xlrd.open_workbook(self.file)
        sheet = workbook.sheet_by_name(self.sheet)
        return sheet
    def GetJenkinsVar(self, key):
        try:
            value = os.environ.get(key)
        except Exception:
            value = os.environ.get(key.upper())
        # print('value = ',value)
        if (not value):
            value = ''
        # print( os.path.join(r'c:/a',value) )
        return value
    def Read_execel(self):
        '''
        返回指定内容在第几行
        :return:
        '''
        Sheet = self.get_values()
        for ncols in range(Sheet.ncols):
            if Sheet.cell_value(0,ncols) == 'pkgname':
                return ncols
    def get_packagename(self):
        '''
        Return all packagename from the execel
        返回execel里的所有包名
        :return:
        '''
        list_package = []
        PNexecel_ncol = self.Read_execel()
        Sheet =self.get_values()
        for nrows in range(1,Sheet.nrows):
            value = Sheet.cell_value(nrows,PNexecel_ncol)
            if value:
                list_package.append(Sheet.cell_value(nrows,PNexecel_ncol))
        return list_package
    def download_httpsapk(self,url,package):
        '''
        download the apk about the way of https
        :return:
        '''
        is_downlaod_success = False
        headers = {"User-Agent": "Mozilla/5.0 (Windows NT 6.1; rv:40.0) Gecko/20100101 Firefox/40.0"}
        for i in xrange(3):
            try:
                req = urllib2.Request(url, headers=headers)
                f = urllib2.urlopen(req, context=ssl.SSLContext(get_protocol()))
                data = f.read()
                with open(os.path.join(self.apkpath,package+".apk"), "wb") as code:
                    code.write(data)
                is_downlaod_success = True
                print package,"下载成功！"
                break
            except:
                if i == 2:
                    print package,"下载失败！"
    def download_httpapk(self,url,package):
        '''
        download the apk about the way of http to Single thread
        :return:
        '''
        is_downlaod_success = False
        test = MyOpener()
        for i in xrange(3):
            try:
                time.sleep(1)
                socket.setdefaulttimeout(300)
                test.retrieve(url,os.path.join(self.apkpath,package+".apk"))
                is_downlaod_success = True
                print package,"下载成功！"
                break
            except:
                if i == 2:
                    print package,"下载失败！请手动下载"
        if not is_downlaod_success:
            print 'download apk is failure'
    def delete_apk(self):
        '''
        删除指定目录下的apk
        :return:
        '''
        try:
            if self.delete_apk != 'false':
                os.system("cd "+self.apkpath+" &del /s /q *.apk")
        except:
            print 'delete is failure'
class link():
    def __init__(self):
        pass
    def find_webpage(self,package):
        '''
        find the web page to screen the link of downloading
        :param package:
        :return:
        '''
        url = "https://m.apkpure.com/"+package+"/"+package+"/download?from=details"
        headers = {"User-Agent": "Mozilla/5.0 (Windows NT 6.1; rv:40.0) Gecko/20100101 Firefox/40.0"}
        req = urllib2.Request(url, headers=headers)
        f = urllib2.urlopen(req, context=ssl.SSLContext(get_protocol()))
        data = f.read()
        link = re.findall('https.*click',data)
        if link:
            links = link[0].replace('">click',"")
            return links
        else:
            print 'The link is failure'
        # with open("test.txt", "wb") as code:
        #     code.write(data)
    def link_wandoujia(self):
        '''
        The link is for wandoujia
        :return:
        '''
        test = download()
        allpackage = test.get_packagename()
        try:
            for i in allpackage:
                link = 'http://www.wandoujia.com/apps/'+i+'/download'
                test.download_httpapk(link,i)
        except:
            print "The link is failure"
    def down_logic(self):
        '''
        Download processing logic
        download the apk from all packages of execel
        :return:
        '''
        test = download()
        allpackage = test.get_packagename()
        for i in allpackage:
            print "开始下载：",i
            try:
                findlink = self.find_webpage(i)
                print "已获取下载链接，下载中"
                if findlink:
                    if 'https' in findlink:
                        test.download_httpsapk(findlink,i)
                    else:
                        print "尝试豌豆荚下载"
                        self.link_wandoujia()
                else:
                    print 'No find the apk for the link'
            except:
                print sys.exc_info(),"Don't find the link for apk"

if __name__ == '__main__':
    test = link()
    test.down_logic()

    # urllib.urlretrieve('http://www.wandoujia.com/apps/com.dianping.v1/binding?source=wandoujia-web_inner_referral_binded',)
