bind/listen(监听数目)
阻塞：accept/connect/rcv/send

自己实现多线程轮询，或者用线程的socketserver的TCPServer和BaseRequestHandler...
select.select可以监听socket对象(或者返沪fileno接口的对象)，返回的对象继续调用socket的接口比如accept/rcv等，然后继续更新select.select的候选列表

```
#! usr/bin/python3
import time
from socket import socket, AF_INET, SOCK_STREAM, SOL_SOCKET, SO_REUSEADDR
from threading import Thread, Lock
from socketserver import TCPServer, ThreadingTCPServer, BaseRequestHandler, StreamRequestHandler
from multiprocessing import Process
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

HOST = "127.0.0.1"
PORT = 50001

def server():
    with socket(AF_INET, SOCK_STREAM) as soc:
        soc.bind((HOST, PORT))
        soc.listen(5)
        soc.setblocking(False)
        list_conn = []
        while True:
            try:
                _new_conn, _new_add = soc.accept()
                print("Server: accept one conn")
                list_conn.append((_new_conn, _new_add))
            except BlockingIOError:
                pass

            for conn, add in list_conn:
                rcv_Idx = 0
                while True:
                    try:
                        data = conn.recv(10)
                        print(f"Server({add}): ---rcv data---: ", type(data), data)
                        rcv_Idx += len(data)
                        if data.endswith(b'\x00'):
                            print("Server: ---rcv end---")
                            conn.send(f"rcv acc: {rcv_Idx}".encode("utf8"))
                    except Exception as e:
                        break


class Client(object):
    def __init__(self, name):
        self.name = name
        self.send_buff = ""
        self.sock = socket(AF_INET, SOCK_STREAM)
        self.lock = Lock()

    def push_buff(self, msg):
        self.send_buff = msg

    def _send(self):
        print(f"Client({self.name}): client send data: {len(self.send_buff.encode('utf8'))}")
        self.sock.sendall(self.send_buff.encode("utf8"))
        data = self.sock.recv(1024)
        print(f"Client({self.name}): ACK: " + data.decode("utf8"))

    def client(self):
        while True:
            if self.send_buff:
                self.lock.acquire()
                self._send()
                self.send_buff = ""
                self.lock.release()

    def __enter__(self):
        print(f"Client({self.name}): enter client and conn socket")
        self.sock.connect((HOST, PORT))

    def __exit__(self, exc_type, exc_val, exc_tb):
        print(f"Client({self.name}): exit client and close socket")
        self.sock.close()

def launch_client():
    cc = Client("cc")
    cc1 = Client("cc1")
    with cc, cc1:
        Thread(target=cc.client).start()
        Thread(target=cc1.client).start()
        while True:
            for c in [cc, cc1]:
                print("action----->")
                c.push_buff(f"CUSTOMER DATA: {time.time()}\0\n")
                time.sleep(3)

def p1():
    Thread(target=server).start()
    launch_client()

def p2():
    class SingleReq(BaseRequestHandler):
        def handle(self):
            while True:
                msg = self.request.recv(1024)
                print("fetch msg", msg)
                if not msg:
                    print("rcv end")
                    self.request.send("ACK")
                    break

    class SingleStreamReq(StreamRequestHandler):
        def handle(self):
            while True:
                try:
                    # for line in self.rfile:  # 阻塞住？？
                    line = self.rfile.read(10)
                    print("fetch msg", line)
                    if not line:
                        print("rcv end")
                        self.wfile.write("ACK")
                        break
                except Exception:
                    pass

    # server = TCPServer((HOST, PORT), SingleReq)
    # server = TCPServer((HOST, PORT), SingleStreamReq)
    server = ThreadingTCPServer((HOST, PORT), SingleStreamReq)

    Thread(target=server.serve_forever).start()
    launch_client()

def event_loop():
    pass

def p3():
    class Handler(object):
        def __init__(self, add):
            self.sock = socket(AF_INET, SOCK_STREAM)
            self.sock.setsockopt(SOL_SOCKET, SO_REUSEADDR, True)
            self.sock.bind(add)
            self.sock.listen(1)

        def fileno(self):
            return self.sock.fileno()

        def handle(self, *args):
            pass

    class HandlerA(Handler):
        def handle(self, *args):
            conn, add = self.sock.accept()
            msg = conn.recv(1024)
            print("HandlerA: rcv: ", msg)

    class HandlerB(Handler):
        def handle(self, *args):
            conn, add = self.sock.accept()
            msg = conn.recv(1024)
            print("HandlerB: rcv: ", msg)

    def event_loop():
        import select
        ha = HandlerA((HOST, PORT))
        hb = HandlerB((HOST, PORT+1))
        while True:
            want_rcvs, want_sends, _ = select.select([ha, hb], [], [])
            for h in want_rcvs:
                h.handle()

    Thread(target=event_loop).start()
    launch_client()


if __name__ == "__main__":
    # p1()
    # p2()
    p3()


```
