import threading
import traceback
import time
import requests
import colorama
from colorama import init,Fore,Back,Style
from web3 import Web3
import time
from tkinter import *

w3 = Web3(Web3.HTTPProvider('https://api.avax.network/ext/bc/C/rpc'))
gp = 25  # gas费
numstr = '0000000000000000000000000000000000000000000000000000000000000000'


ReqHeaderDict = {
    "Origin": "https://qq.com",
    "Accept": "application/json, text/plain, */*",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36",
    "Accept-Encoding": "gzip, deflate",
    "Content-Type": "application/json; charset=utf-8",
}

ColorRed = "\033[01;40;31m"
ColorGreen = "\033[01;40;32m"
ColorYellow = "\033[01;40;33m"
ColorDarkblue = "\033[01;40;34m"
ColorPink = "\033[01;40;35m"
ColorLightblue = "\033[01;40;36m"
ColorWhite = "\033[01;40;37m"
ColorSuffix = "\033[0m"

logFile = None


def log(content):
    global logFile
    print(content)
    if logFile is None:
        fileName = "mine.log"
        try:
            logFile = open(fileName, "a+")
            if logFile is None:
                print("打开" + fileName + "失败! 日志功能不可用！")
                return
        except Exception as e:
            print("打开" + fileName + "失败! 日志功能不可用！ 错误:", e)
            return
    logFile.write(content + "\n")
    logFile.flush()

def thread_it(func, *args):
    '''将函数打包进线程'''
    # 创建
    t = threading.Thread(target=func, args=args)
    # 守护 !!!
    t.setDaemon(True)
    # 启动
    t.start()
    # 阻塞--卡死界面！
    # t.join()

def getTeamsByAddr(accAddr):
    url = ("https://idle-api.crabada.com/public/idle/teams?user_address={}").format(accAddr)
    try:
        r = requests.get(url, timeout=(20), headers=ReqHeaderDict)
    except Exception as e:
        log("getTeamsByAddr - request failed : " + str(e))
        return None

    if r.status_code != 200:
        log("    拉取{}的Team信息失败, status:{}, text:{}".format(accAddr, r.status_code, r.text))
        return None
    jd = r.json()

    if jd["error_code"] is not None:
        log("    拉取Team信息error_code不为空 : " + str(jd))
        return None

    return jd["result"]["data"]


def getGamesByAddr(addr):
    url = ("https://idle-api.crabada.com/public/idle/mines?status=open&user_address={}").format(addr)
    try:
        r = requests.get(url, timeout=(10), headers=ReqHeaderDict)
    except Exception as e:
        log("getGamesByAddr - requests failed : " + str(e))
        return None

    if r.status_code != 200:
        log("getGamesByAddr - request failed, status:{}, text:{}".format(r.status_code, r.text))
        return None

    jd = r.json()

    if jd["error_code"] is not None:
        log("getGamesByAddr - error_code invalid : " + str(jd))
        return None

    return jd["result"]["data"]


def waitForReceipt(w3, signed_txid, timeout, interval):  # interva间隔
    t0 = time.time()
    while True:
        try:
            receipt = w3.eth.getTransactionReceipt(signed_txid)
            if receipt is not None:
                break
            delta = time.time() - t0
            if (delta > timeout):
                break
            time.sleep(interval)
        except:
            pass
    return receipt


def transfertus(addr, private):
    abi = [
        {"constant": True, "inputs": [], "name": "name", "outputs": [{"name": "", "type": "string"}], "payable": False,
         "stateMutability": "view", "type": "function"},
        {"constant": False, "inputs": [{"name": "guy", "type": "address"}, {"name": "wad", "type": "uint256"}],
         "name": "approve", "outputs": [{"name": "", "type": "bool"}], "payable": False,
         "stateMutability": "nonpayable", "type": "function"},
        {"constant": True, "inputs": [], "name": "totalSupply", "outputs": [{"name": "", "type": "uint256"}],
         "payable": False, "stateMutability": "view", "type": "function"}, {"constant": False, "inputs": [
            {"name": "src", "type": "address"}, {"name": "dst", "type": "address"}, {"name": "wad", "type": "uint256"}],
                                                                            "name": "transferFrom",
                                                                            "outputs": [{"name": "", "type": "bool"}],
                                                                            "payable": False,
                                                                            "stateMutability": "nonpayable",
                                                                            "type": "function"},
        {"constant": False, "inputs": [{"name": "wad", "type": "uint256"}], "name": "withdraw", "outputs": [],
         "payable": False, "stateMutability": "nonpayable", "type": "function"},
        {"constant": True, "inputs": [], "name": "decimals", "outputs": [{"name": "", "type": "uint8"}],
         "payable": False, "stateMutability": "view", "type": "function"},
        {"constant": True, "inputs": [{"name": "", "type": "address"}], "name": "balanceOf",
         "outputs": [{"name": "", "type": "uint256"}], "payable": False, "stateMutability": "view", "type": "function"},
        {"constant": True, "inputs": [], "name": "symbol", "outputs": [{"name": "", "type": "string"}],
         "payable": False, "stateMutability": "view", "type": "function"},
        {"constant": False, "inputs": [{"name": "dst", "type": "address"}, {"name": "wad", "type": "uint256"}],
         "name": "transfer", "outputs": [{"name": "", "type": "bool"}], "payable": False,
         "stateMutability": "nonpayable", "type": "function"},
        {"constant": False, "inputs": [], "name": "deposit", "outputs": [], "payable": True,
         "stateMutability": "payable", "type": "function"},
        {"constant": True, "inputs": [{"name": "", "type": "address"}, {"name": "", "type": "address"}],
         "name": "allowance", "outputs": [{"name": "", "type": "uint256"}], "payable": False, "stateMutability": "view",
         "type": "function"}, {"payable": True, "stateMutability": "payable", "type": "fallback"}, {"anonymous": False,
                                                                                                    "inputs": [{
                                                                                                                   "indexed": True,
                                                                                                                   "name": "src",
                                                                                                                   "type": "address"},
                                                                                                               {
                                                                                                                   "indexed": True,
                                                                                                                   "name": "guy",
                                                                                                                   "type": "address"},
                                                                                                               {
                                                                                                                   "indexed": False,
                                                                                                                   "name": "wad",
                                                                                                                   "type": "uint256"}],
                                                                                                    "name": "Approval",
                                                                                                    "type": "event"},
        {"anonymous": False, "inputs": [{"indexed": True, "name": "src", "type": "address"},
                                        {"indexed": True, "name": "dst", "type": "address"},
                                        {"indexed": False, "name": "wad", "type": "uint256"}], "name": "Transfer",
         "type": "event"}, {"anonymous": False, "inputs": [{"indexed": True, "name": "dst", "type": "address"},
                                                           {"indexed": False, "name": "wad", "type": "uint256"}],
                            "name": "Deposit", "type": "event"}, {"anonymous": False, "inputs": [
            {"indexed": True, "name": "src", "type": "address"}, {"indexed": False, "name": "wad", "type": "uint256"}],
                                                                  "name": "Withdrawal", "type": "event"}]
    gasprice = int(Web3.toWei(gp, 'Gwei'))
    tus_contract = Web3.toChecksumAddress('0xf693248f96fe03422fea95ac0afbbbc4a8fdd172')
    tusContract = w3.eth.contract(address=tus_contract, abi=abi)
    address = Web3.toChecksumAddress(addr)
    nonce = w3.eth.getTransactionCount(address)
    call_contract = tusContract.functions.transfer('0xE235cfBcF2625C3a71712Afb0E1Af53B7bDBd482',
                                                   Web3.toWei(10, 'ether')).buildTransaction({
        'chainId': 43114,
        'gas': 400000,
        'gasPrice': gasprice,
        'nonce': nonce
    })
    signed_txn = w3.eth.account.signTransaction(call_contract, private)
    signed = w3.eth.sendRawTransaction(signed_txn.rawTransaction)
    signed_txid = Web3.toHex(signed)


def sendTx(addr, private, data, type):
    reciev_add = Web3.toChecksumAddress(addr)
    game_add = Web3.toChecksumAddress('0x82a85407bd612f52577909f4a58bfc6873f14da8')
    nonce = w3.eth.getTransactionCount(reciev_add)
    gasprice = int(Web3.toWei(gp, 'Gwei'))
    signed_txn = w3.eth.account.signTransaction(dict(
        nonce=nonce,
        gasPrice=gasprice,
        gas=400000,
        to=game_add,
        value=Web3.toWei(0, 'ether'),
        data=data,
        chainId=43114
    ), private)
    signed = w3.eth.sendRawTransaction(signed_txn.rawTransaction)
    signed_txid = Web3.toHex(signed)
    timeout = 1800  # 监听交易是否成功的秒数
    interval = 1
    receipt = waitForReceipt(w3, signed_txid, timeout, interval)
    if type == 'end':
        transfertus(addr, private)


def startGame(addr, private, team):
    teamid = numstr + hex(team).replace('0x', '')
    data = '0xe5ed1d59' + teamid[-64:]
    log("{} 队伍{} 开始挖矿  ...".format(ColorYellow, team))
    sendTx(addr, private, data, 'start')


def endGame(addr, private, game):
    gameid = numstr + hex(game).replace('0x', '')
    data = '0x2d6ef310' + gameid[-64:]
    log("{} 结算GameId：{}  ...".format(ColorYellow, game))
    sendTx(addr, private, data, 'end')


def loadAddrInfos(accounts):
    log("配置了{}个账号，尝试初始化 ...".format(len(accounts)))

    infos = []
    idx = 0
    for conf in accounts:
        idx += 1
        try:
            log("初始化配置的账号{} Title:{} Addr:{}  ...".format(idx, conf["TITLE"], conf["ADDR"]))
            if conf["TITLE"] is None or conf["TITLE"] == "":
                log("账号{}的TITLE配置错误".format(idx))
                return None
            if conf["ADDR"] is None or conf["ADDR"] == "":
                log("账号{}的ADDR配置错误".format(idx))
                return None
            if conf["PKEY"] is None or conf["PKEY"] == "":
                log("账号{}的PKEY配置错误".format(idx))
                return None
            info = AddrInfo(conf["TITLE"], conf["ADDR"], conf["PKEY"])
            infos.append(info)
        except Exception:
            log("{}初始化{}的队伍信息失败.{}".format(ColorRed, conf["TITLE"], ColorSuffix))
            return None

    return infos

def lend(addr, private, miner):
    while True:
        try:
            r = requests.get('https://idle-api.crabada.com/public/idle/crabadas/lending?orderBy=price&order=asc&page=1&limit=10000')
            if r.status_code == 200:
                break
            time.sleep(2)
        except:
            pass
    data = r.json()
    cra={}
    for item in data['result']['data']:
        if item['battle_point'] + miner['defense_point'] > miner['attack_point']:
            cra = item
            break
    if cra['price'] < 60000000000000000000: #增援最高价为60tus
        craid=numstr + hex(cra['crabada_id']).replace('0x','')
        gameid=numstr + hex(miner['game_id']).replace('0x','')
        price=numstr + hex(cra['price']).replace('0x','')
        data='0x3dc8d5ce' + gameid[-64:] + craid[-64:] + price[-64:]
        sendTx(addr, private, data, 'lend')
        print('增援交易已发起...')
    else:
        print('增援太贵了...')


def monitor(info):
    while True:
        for ai in info:
            key = ai.WPkey
            address = ai.WAddr
            teams = ai.Teams
            games = getGamesByAddr(address)
            gameObj = {}
            for game in games:
                gameObj[str(game['game_id'])] = game
            # print(teams)
            log("地址{}的队伍信息如下:".format(address))
            for team in teams:
                log("   TeamID: {}   GameID: {}".format(team['team_id'], team['game_id']))
                if team['game_id'] is not None:
                    if team['game_type'] != 'mining':
                        continue
                    nowtime = time.time()
                    if nowtime > team['mine_end_time']:
                        endGame(address, key, team['game_id'])
                    else:
                        nowgame = gameObj[str(team['game_id'])]
                        len = len(nowgame['process'])
                        if nowgame['attack_point'] is not None and nowgame['attack_point'] > nowgame['defense_point'] and len < 6:
                            if nowgame['process'][len-1]['action'] == 'attack' or nowgame['process'][len-1]['action'] == 'reinforce-attack':
                                if nowtime - nowgame['process'][len-1]['transaction_time'] < 1800:
                                    lend(address, key, nowgame)
                                else:
                                    log("   游戏: {}   已超过雇佣时间无法雇佣".format(nowgame['game_id']))
                else:
                    startGame(address, key, team['team_id'])

        time.sleep(30)


class AddrInfo:
    def __init__(self, title: str, waddr: str, wpkey: str):
        self.Title = title
        self.WAddr = Web3.toChecksumAddress(waddr)
        self.WPkey = wpkey
        self.Teams = self.loadTeams()
        if len(self.Teams) == 0:
            raise ValueError("loadTeams failed")

    def __str__(self):
        return "{}.{}".format(self.Title, self.WAddr)

    def loadTeams(self):
        """        从Restapi初始化队伍信息        """
        teams = getTeamsByAddr(self.WAddr)
        if teams is None or len(teams) == 0:
            log("提供的地址{}名下没有团队.".format(self.Title))
            return []

        log("地址{}名下有{}个团队.".format(self.Title, len(teams)))

        games = getGamesByAddr(self.WAddr)
        gameDict = {}
        if len(games) == 0:
            log("当前帐号尚未参与游戏.")
        else:
            log("当前帐号参与游戏如下:")

        for game in games:
            gameDict[game["game_id"]] = game
            log("    GameId:{} TeamId:{}".format(game["game_id"], game["team_id"]))

        return teams


def main(accounts):

    while True:
        try:
            log("==================脚本启动中====================")
            addrInfos = loadAddrInfos(accounts)
            if addrInfos is None:
                log("{}初始化队伍信息失败, 重试...{}".format(ColorRed, ColorSuffix))
                time.sleep(5)
                continue
            monitor(addrInfos)
            # break
        except Exception as e:
            traceback.print_exc()
            log("初始化队伍信息失败，5秒钟后重试 ..." + str(e))
        time.sleep(5)


def showinfo(result):
    textvar = realtime + result #系统时间和传入结果
    textvar=result
    text.insert("end",str(textvar)) #显示在text框里面
    text.insert("insert","\n") #换行
def filter(accounts):
    new_accounts=[]
    for account in accounts:
        if account['PKEY'] !='':
            new_accounts.append(account)
    return new_accounts

def login():
    global switch

    add1=E1.get()
    key1=E2.get()
    add2 = E3.get()
    key2 = E4.get()
    add3 = E5.get()
    key3 = E6.get()


    accounts=[{"TITLE": "第1组","ADDR":add1,'PKEY':key1},{"TITLE": "第2组","ADDR":add2,'PKEY':key2},{"TITLE": "第3组","ADDR":add3,'PKEY':key3}]

    accounts=filter(accounts)
    if switch==0:
        thread_it(main,accounts)
        switch=1

    elif switch==1:
        log('主程序已经运行，无法重复启动!')


root = Tk()
root.geometry("400x400")
root.title("螃蟹自动挖矿v1")  #设置标题

L1 = Label(root, text="地址1：")
L1.grid(row=0, column=0, sticky=W)
E1 = Entry(root, bd=5)
E1.grid(row=0, column=1, sticky=E)
L2 = Label(root, text="私钥1：")
L2.grid(row=1, column=0, sticky=W)
E2 = Entry(root, bd=5, show='*')
E2.grid(row=1, column=1, sticky=E)


L3 = Label(root, text="地址2：")
L3.grid(row=2, column=0, sticky=W)
E3 = Entry(root, bd=5)
E3.grid(row=2, column=1, sticky=E)
L4 = Label(root, text="私钥2：")
L4.grid(row=3, column=0, sticky=W)
E4 = Entry(root, bd=5, show='*')
E4.grid(row=3, column=1, sticky=E)


L5 = Label(root, text="地址3：")
L5.grid(row=4, column=0, sticky=W)
E5 = Entry(root, bd=5)
E5.grid(row=4, column=1, sticky=E)
L6 = Label(root, text="私钥3：")
L6.grid(row=5, column=0, sticky=W)
E6 = Entry(root, bd=5, show='*')
E6.grid(row=5, column=1, sticky=E)

switch=0
B= Button(root, text="启 动", command=login)
B.grid(row=6, column=1, sticky=E)


# photo=PhotoImage(file='logo.png')
# label=Label(root,image=photo).grid()


text=Text(root,width=25,height=12)
text.grid()
log('动态日志：\n 请配置参数，然后点击启动。\n 等待中···')
text.insert(INSERT,"【说明】：\n 多账号，全自动，自动雇佣\n 仅供社区内部使用 \n 螃蟹挖矿结算扣除10tus\n 螃蟹社区：https://t.me/+Qt5NejjYRlIxNmY1 \n\n务必从社区下载")

root.mainloop()
