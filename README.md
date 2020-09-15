import time
import socket


class mt1000a:
    def __init__(self, ip, timeout=15):
        try:
            self.ip = ip
            self.soc_1000a = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            self.soc_1000a.settimeout(timeout)
            tcpport = 56001
            self.soc_1000a.connect((self.ip, tcpport))
            # cmd = "SYST:PROM 1"
            # self.sendCmd(cmd)
        except Exception as e:
            print("Error: " + str(e))

    def sendCmd(self, cmd):
        try:
            # returns str, gets str parameter
            cmd = cmd + '\n'
            print(cmd)
            self.soc_1000a.send(cmd.encode())
            time.sleep(1)
            cmd2 = ":SYSTEM:ERROR?"
            error = self.reqData(cmd2)
            try:
                if int(error[0]) != 0:
                    print(" !", error)
            except:
                pass
            return error
        except Exception as error:
            print("Error: " + str(error))
            return error

    def reqData(self, cmd, timeout=15):
        #gets str type, returns str parameter
        cmd = cmd + '\n'
        self.soc_1000a.settimeout(timeout)
        print(cmd)
        # Protect from timeout
        try:
            self.soc_1000a.send(cmd.encode())
            time.sleep(1)
            data = self.soc_1000a.recv(1024)
            data = data.decode('UTF-8', 'ignore')
            data = data.rstrip("\n")
            data = data.strip('"')
            print(data)
            return data
        except:
            print("  ! Timeout on request " + cmd.rstrip('\n'))
            data = '99997'
        if data == '9.91e+37':
            print(" !Result not applicable")
        elif data == '9.9e+37':
            print(' !Result overrange')
            data = '99999'
        elif data == '- 9.91e+37':
            print(" ! Result underrange")
            data = '-99999'
        return data

    def toggleTrafficON(self):
        cmd = "ETH:TRAF:GEN:STAR"
        self.sendCmd(cmd)
        cmd = "MEAS:STAR"
        self.sendCmd(cmd)

    def toggleTrafficOFF(self):
        cmd = "MEAS:STOP"
        self.sendCmd(cmd)
        time.sleep(1)
        cmd = "ETH:TRAF:GEN:STOP"
        self.sendCmd(cmd)


    def config_bert_eth_1gb(self):
        cmd = "SYST:PROM 1"
        self.sendCmd(cmd)
        time.sleep(1)
        cmd = "*RST"
        self.sendCmd(cmd)
        time.sleep(2)
        cmd = "INST:STAR TP-BERT-ETH,1-PORT1,1-PORT2"
        self.sendCmd(cmd)
        time.sleep(6)
        cmd = "ETH:PORT1:ITYP SFPP"
        self.sendCmd(cmd)
        time.sleep(7)
        cmd = "ETH:PORT2:ITYP SFPP"
        self.sendCmd(cmd)
        time.sleep(7)
        cmd = "ETH:PORT1:MODE FORC"
        self.sendCmd(cmd)
        time.sleep(3)
        cmd = "ETH:PORT2:MODE FORC"
        self.sendCmd(cmd)
        time.sleep(1)
        cmd = "ETH:PORT1:TRAF:STR1:LL 10.0000"
        self.sendCmd(cmd)
        time.sleep(1)
        cmd = "ETH:PORT2:TRAF:STR1:LL 10.0000"
        self.sendCmd(cmd)
        time.sleep(1)
        cmd = "MEAS:SET:ILEN 1S" #interval length
        self.sendCmd(cmd)
        time.sleep(1)
        cmd = 'ETH:PORT1:STR1:MAC:SOUR "00-50-C2-35-D2-EF"'
        self.sendCmd(cmd)
        time.sleep(1)
        cmd = 'ETH:PORT2:STR1:MAC:DEST "00-50-C2-35-D2-EF"'
        self.sendCmd(cmd)
        time.sleep(1)
        cmd = "ETH:PORT1:SETT:ASTG ON"
        self.sendCmd(cmd)
        time.sleep(1)
        cmd = 'ETH:PORT2:SETT:ASTG ON'
        self.sendCmd(cmd)
        time.sleep(1)
        cmd = 'INST:STAR:GUI'
        self.sendCmd(cmd)

    def bert_check(self):
        # cmd = "SYST:PROM 0"
        # self.sendCmd(cmd)
        inst = mt1000a.reqData("INST?")
        if inst == '-1':
            self.sendCmd("INST:CONN:ALL")
            time.sleep(1)
            type1 = mt1000a.reqData("ETH:PORT1:ITYP?")
            time.sleep(2)
            type2 = mt1000a.reqData("ETH:PORT2:ITYP?")
            if type1 == type2 == "SFPP":
                print("Pass")


        time.sleep(2)
        # cmd = '*CLS'
        #self.sendCmd(cmd)
        # cmd = "ETH:PORT1:ITYP?"
        # self.sendCmd(cmd)
        # time.sleep(1)
        # type = mt1000a.reqData("ETH:PORT1:ITYP?")
        # time.sleep(3)
        # mt1000a.reqData("ETH:PORT2:ITYP?")


mt1000a = mt1000a("192.168.200.5")
mt1000a.bert_check()
# mt1000a.config_bert_eth_1gb()
# time.sleep(10)
# mt1000a.toggleTrafficON()
# time.sleep(20)
# mt1000a.reqData("ETH:PORT1:IFET? (BPE)") #Pattern Errors
# time.sleep(1)
# mt1000a.reqData("ETH:PORT1:IFET? (BSE)") #Sequence Errors
# time.sleep(1)
# mt1000a.reqData("ETH:PORT1:IFET? (BSSL)") #Sequence Sync
# time.sleep(1)
# mt1000a.reqData("ETH:PORT1:IFET? (BFL)") #Frame Loss
# time.sleep(1)
# mt1000a.reqData("ETH:PORT1:IFET? (BFLS)") #Frame Loss seconds
# time.sleep(1)
# mt1000a.reqData("ETH:PORT1:IFET? (LOS)") #Loss of signal
# time.sleep(1)
# mt1000a.reqData("ETH:PORT1:IFET? (TEFT)") #Total Errored frames
# time.sleep(1)
# mt1000a.reqData("ETH:PORT1:IFET? (FFR)") #Fragmented frames
# time.sleep(1)
# mt1000a.reqData("ETH:PORT1:IFET? (FEFR)") #FCS Errored frames
# mt1000a.toggleTrafficOFF()
# time.sleep(1)
# cmd = '*CLS'
# mt1000a.sendCmd(cmd)



