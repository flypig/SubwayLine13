"""
@auther: chenhaichao@baidu.com
@date  : 2012/12/23

@note  : stay hungry stay foolish
"""


import threading
import socket
import struct
import ConfigParser
from optparse import OptionParser
import string
import sys,os
import hashlib
import json
from baselib.Packlib.ulpack import *
import baselib.Packlib.spider_mcpack as mcpack
import baselib.Packlib.packNetReceiver as packNetReceiver
import baselib.Packlib.packNetSender as packNetSender


"""
global varies
"""

def getWeiboUlpack(item):
    packet = ULPacket()
    packet["Url"]="http://weibo.com/" + item.weiboId
    packet["Method"]="ADD"
    packet["InTime"]=item.time
    packet["Last-mod-time"]=item.time
    packet["Createtime"]=item.time
    packet["LinkFoundTime"]=item.time
    packet["Codetype"]="GB"
    packet["Sexfactor"]="0"
    packet["Politicalfactor"]="0"

    try:
        talk_unicode=item.talk.decode('utf-8')
        talk_filter=''.join([c for c in talk_unicode if c != u'\xa0'])
        talk_gbk=talk_filter.encode('gbk')
    
        xml_unicode=item.xmlData().decode('utf-8')
        xml_filter=''.join([ c for c in xml_unicode if c != u'\xa0'])
        xml_gbk=xml_filter.encode('gbk')
    
        header="TITL0+CONT"+str(len(talk_gbk))+"+PAGE0+"+"META"+str(len(xml_gbk))
        packet["Body"]=header+"\r\n\r\n"+talk_gbk+xml_gbk
        return packet
    except:
        print "Unicode To GBK Error"


def TransMcDict2Dict(McDict):

    ResultDict={}
    TrespassFieldDict={}
    spiderDict={}

    for Treskey in McDict:
        if Treskey == 'trespassing_field':
            pass
        else:
            ResultDict[Treskey] = McDict[Treskey]
    for R_key, R_value in ResultDict.items():
        ResultDict[R_key] = R_value
    
    return ResultDict




def handleOnepack(pack):
    print "get a packet"
    Mcpack = repr(pack)
    nshead_size=struct.calcsize('HHI16sIII')
    format = "HHI16sIII%ds" % (len(Mcpack) - nshead_size)
    (id, \
     version, \
     log_id, \
     provider, \
     magic_num,\
     reserved, \
     body_len, \
     body) = struct.unpack(format, Mcpack)

    dict=TransMcDict2Dict(mcpack.binary2dict_old(body))
    print dict['cur_url']

class WeiboEcServer:
    def __init__(self, port, saveWeiboFile, saveAuthorFile, bailingSock):
        self.port=port
        self.mcpackReceiver=None
        self.saveWeiboFile=saveWeiboFile
        self.saveAuthorFile=saveAuthorFile
        self.sendSock=bailingSock

    def start(self):
        mcpackReceiver = packNetReceiver.McpackReceiver(port)
        mcpackReceiver.handleOnepack = handleOnepack
        mcpackReceiver.start()

   


if __name__ == "__main__":
    usage="python2.7 WeiboEC.py [-c <config file>] [-h] [-v]"
    optParser=OptionParser(usage)
    optParser.add_option("-c","--configure",action="store", type="string", dest="configFile" , help="configure file")
    options, args=optParser.parse_args(sys.argv)

    if not options.configFile:
        print usage
        exit(-1)

    config=ConfigParser.RawConfigParser()
    config.read(options.configFile)
    port=config.getint('main','port')
    saveWeiboPath = config.get('main','save_weibo_path')
    saveAuthorPath = config.get('main', 'save_author_path')
    bailingHost= config.get('main','bailing_host')
    bailingPort= config.get('main', 'bailing_port')

    

    while True:
        time.sleep(1)

    """
    parseWeiboHtml(html) 
    authorInfo.printAuthorInfo()
    for item in allInfoList:
        item.printWeiboItem()
    #saveUlpackToFile()
    """
    
