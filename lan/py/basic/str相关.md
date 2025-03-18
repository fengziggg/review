字符串相关  
关注正则的基础用法, match/findall/finditer/捕获组，最短匹配，跨行匹配，替换sub  
unicode相关的练手，但unicode的知识要梳理补充...  

字符串格式化相关:format_map，字符串和字节串的梳理，令牌流练手re  

```
#! usr/bin/python3
# -*- encoding:utf-8 -*-
import re

def splite_out(func):
    def wrap(*args):
        name = func.__name__
        print(name.center(20, "-"))
        func()
        print()
    return wrap

@splite_out
def practice_2_1():
    # practice re
    ss = "asdf fjdk; afed, fjek,asdf, foo"
    print(re.split(r"[;,\s]\s*", ss))
    value = re.split(r"(;|,|\s)\s*", ss)  # 捕获分组
    print(value[::2])
    print(value[1::2])
    value2 = re.split(r"(?:;|,|\s)\s*", ss)  # 转化为非捕获
    print(value2)

@splite_out
def practice_2_2():
    filename = "spam.txt"
    print(filename.endswith(".txt"))
    print(filename.startswith("file:"))
    print(filename.startswith("spam"))
    import os
    filenames = os.listdir(".")
    print(filenames)
    print(any(name.endswith(".py") for name in filenames))
    from urllib.request import urlopen
    url = "http://www.python.org"
    # if url.startswith(("http:", "https:", "ftp:")):
    #     print(urlopen(url).read())
    # else:
    #     print("no match")
    import re
    print(re.match(r"http:|https:|ftp:/s*", url))
    
@splite_out
def practice_2_3():
    # find/match/findall/finditer/捕获
    text1 = '11/27/2012'
    text2 = 'Nov 27, 2012'
    print(text1.find("Nov"))
    print(text2.find("Nov"))
    
    import re
    print(re.match(r"\d+/\d+/\d+", text1))
    print(re.match(r"\d+/\d+/\d+", text2))
    
    text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
    m = re.compile(r"\d+/\d+/\d+")
    print(m.match(text1))  # 严格从序列0开始匹配
    print(m.match(text2))
    print(m.match(text))
    print(m.findall(text))
    for mm in m.findall(text):
        print(mm, type(mm))
    print(m.finditer(text))
    for mmm in m.finditer(text):
        print(mmm, type(mmm))

    m = re.compile(r"(\d+)/(\d+)/(\d+)")
    mm = m.match("11/27/2012")
    print(mm.groups(), mm.group(0), mm.group(1), mm.group(1), mm.group(2))

@splite_out
def practice_2_4():
    # re sub用法，在compile后默认match的基础上rpl
    import re
    text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
    print(text.replace("Today", "Yesterday"))
    m = re.compile(r"(?P<mon>\d+)/(?P<day>\d+)/(?P<year>\d+)")
    objs = m.finditer(text)
    for obj in objs:
        print(obj.groups())
    print(m.sub(r"\3-\1-\2", text))
    print(m.sub(r"\g<year>-\g<mon>-\g<day>", text))
    print(m.sub(r"RPL_DATE", text))
    ss, n = m.subn(r"\g<year>-\g<mon>-\g<day>", text+"1/1/1")
    print(n)

@splite_out
def practice_2_5():
    text = 'UPPER PYTHON, lower python, Mixed Python'
    import re
    print(re.findall("python", text, flags=re.IGNORECASE))
    print(re.sub("python", "snake", text, flags=re.IGNORECASE))
    def wrap(target):
        def _wrap(m):
            if m.group(0).islower():
                return target.lower()
            elif m.group().isupper():
                return target.upper()
            elif m.group()[0].isupper():
                return target.capitalize()
            return target
        return _wrap
    print(re.sub("python", wrap("snake"), text, flags=re.IGNORECASE, ))

@splite_out
def practice_2_6():
    # 最短匹配，捕获组内和外都会进行match，？是作用于捕获组外提示下有多个捕获组的时候分割最小捕获组
    text2 = 'Computer says "no." Phone says "yes."'
    import re
    m1 = re.compile(r'(".*")')
    m2 = re.compile(r'(".*"?)')
    print(m1.findall(text2))   
    print(m2.findall(text2))
    m3 = re.compile(r'"(.*)"')
    m4 = re.compile(r'"(.*?)"')
    print(m3.findall(text2))
    print(m4.findall(text2))

@splite_out
def practice_2_7():
    # 跨行匹配，就是吧\n纳入.的匹配集合，然后多余的空字符用非限定解决
    text1 = '''/* this is a single comment */'''
    text2 = '''/* this is a
 multiline comment */
'''
    import re
    m1 = re.compile(r'/\*(.*?)\*/')
    print(m1.findall(text1))
    print(m1.findall(text2), end="\n\n")
    m2 = re.compile(r"/\*((?:.|\n)*?)\*/")
    print(m2.findall(text1))
    print(m2.findall(text2), end="\n\n")
    m3 = re.compile(r"/\*((.|\n)*?)\*/")  # 非限定的.会吧空字符串也包含
    print(m3.findall(text1))
    print(m3.findall(text2), end="\n\n")
    m4 = re.compile(r"/\*(.*?)\*/", flags=re.DOTALL)
    print(m4.findall(text2))
    
@splite_out
def practice_2_8():
    s1 = 'Spicy Jalape\u00f1o'
    s2 = 'Spicy Jalapen\u0303o'
    import unicodedata
    print(s1)
    print(s2, end="\n\n")
    t1 = unicodedata.normalize("NFC", s1)
    t2 = unicodedata.normalize("NFC", s2)
    print(t1)
    print(t2)
    print(t2 == t1, end="\n\n")
    t3 = unicodedata.normalize("NFD", s1)
    t4 = unicodedata.normalize("NFD", s2)
    print(t3)
    print(t4)
    print(t3 == t4, end="\n\n")
    t5 = "".join(c for c in t3 if not unicodedata.combining(c))
    print(t5)
    t6 = "".join(c for c in t4 if not unicodedata.combining(c))
    print(t6)

@splite_out
def practice_2_9():
    import re
    s = ' hello world \n'
    t = '-----hello====='
    print(s.strip())
    print(s.rstrip())
    print(s.lstrip())
    print(t.strip("-="))
    print(t.rstrip("-="))
    print(t.lstrip("-="))

    print(s.replace(" ", ""))
    print(re.sub(r"\s+", " ", s))

@splite_out
def practice_2_10():
    # translate, chr是unicode对应的值，ord是对应的Unicode字符？？
    s = 'pýtĥöñ\fis\tawesome\r\nhaha'
    print(s)
    emptymap = {
        ord('\n'): " ",
        ord('\f'): " ",
        ord('\r'): None,
    }
    ss = s.translate(emptymap)
    print(ss)
    import sys
    import unicodedata
    charmap = dict.fromkeys(c for c in range(sys.maxunicode) if unicodedata.combining(chr(c)))
    print(ord("a"), type(ord("a")), chr(97))
    print(charmap, len(charmap), sys.maxunicode)
    print(unicodedata.normalize("NFD", ss).translate(charmap))


@splite_out
def practice_2_11():
    # format
    text = 'Hello World'
    print(text.ljust(20, "="))
    print(text.rjust(20, "="))
    print(text.center(20, "="))
    print(format(text, "*>20s"))
    print(format(text, "*<20s"))
    print(format(text, "*^20s"))
    print("{:>20s} {:*^20s}".format("HELLO", "WORLD"))
    print(format(20, "-^20"))  # 格式提示符的s是字符串，非字符串不加
    print("%20s %10s" %("AA", "BB"))

@splite_out
def practice_2_12():
    # concate
    parts = ['Is', 'Chicago', 'Not', 'Chicago?']
    print(" ".join(parts))
    print("CHicago" + " Is" + " Not" + " Chicago?")
    print("Chicago" "Is" "Not" "Chicago")
    print("Chicago", "Is", "not", "chicago", sep=",", end="\n\n")
    
    s = ""
    for w in parts:
        s += w  # 性能问题，一直重创
    print(s)
    def concat():
        for w in parts:
            yield w
    print(" ".join(concat()), end="\n\n")
    
@splite_out
def practice_2_13():
    # format map
    s = '{name} has {n} messages.'
    ss = '{name} has {n} messages.{ohter}'
    name = "Guido"
    n = 28
    class Obj(object):
        def __init__(self):
            self.name = "Obj Guido"
            self.n = 29
    class MapError(dict):  # 是个dict，且直接构造包围目标
        def __missing__(self, key):
            return "{miss: " + key + "}"

    obj = Obj()
    print(s.format(name="GUIDO", n=28))
    print(s.format_map(vars()))
    print(s.format_map(vars(obj)))
    print(ss.format_map(MapError(vars(obj))))
    import sys
    ohter = "hhah"
    print(ss.format_map(MapError(sys._getframe().f_locals)), end="\n\n")  # f_local是cache
    
    print("%(name)s %(n)s" %(vars()))
    import string
    print(string.Template("TmpObj: $name is $n").substitute(vars()))
    

@splite_out
def practice_2_14():
    # 文本格式控制
    import textwrap
    s = "Look into my eyes, look into my eyes, the eyes, the eyes, \
the eyes, not around the eyes, don't look around the eyes, \
look into my eyes, you're under."
    print(s)
    print(textwrap.fill(s, 70), end="\n\n")
    print(textwrap.fill(s, 40, initial_indent="    "), end="\n\n")
    print(textwrap.fill(s, 40, subsequent_indent="-"), end="\n\n")
    import os
    # print(os.get_terminal_size().columns)

@splite_out
def practice_2_15():
    #  re 令牌,
    # 普通查找存在用match，迭代查(不结束)用scanner
    # nametuple是可以用命名访问的元组
    # 扫描字符串解析字符(不一定是单个)为一个个包含字符内容，类型等数据的节点
    text = 'foo = 23 + 42 * 10'

    NAME = r"(?P<NAME>[A-Za-z][0-9A-Za-z_]*)"
    VAL = r"(?P<VALUE>[0-9]+)"
    EQ = r"(?P<EQ>=)"
    TIME = r"(?P<TIME>\*)"
    PLUS = r"(?P<PLUS>\+)"
    WS = r"(?P<WS>\s+)"
    LE = r"(?P<LE><=)"  # 匹配优先度
    GE = r"(?P<GE>>=)"
    LT = r"(?P<LT><)"
    GT = r"(?P<GT>>)"
    import re
    pat = re.compile("|".join([NAME, VAL, EQ, TIME, PLUS, WS, LT, GT]))
    sc = pat.scanner(text)
    _ = sc.match()  # 是个生成器？？
    print(_, _.lastgroup, _.group())
    _ = sc.match()
    print(_, _.lastgroup, _.group())
    for m in iter(sc.match, None):  # 直接迭代化
        print(m.lastgroup, m.group())
        
    from collections import namedtuple
    def gen_token(sc):
        Token = namedtuple("Token", ["Type", "Value"])
        for m in iter(sc.match, None):
            yield Token(m.lastgroup, m.group())

    sc = pat.scanner("foo = 42")
    for t in gen_token(sc):
        print(t, t._fields)

@splite_out
def practice_2_16():
    #  简答递归下降解析器
    # 读取每个令牌(一般是单个字符)，按优先操作解析令牌：取值，乘法，加法
    # 乘法贪婪计算每次处理两个操作数，加法只用乘法的结果做计算
    # 加法嵌套乘法，乘法嵌套取值
    NAME = r"(?P<NAME>[A-Za-z][0-9A-Za-z_]*)"
    VAL = r"(?P<VALUE>[0-9]+)"
    EQ = r"(?P<EQ>=)"
    TIME = r"(?P<TIME>\*)"
    PLUS = r"(?P<PLUS>\+)"
    WS = r"(?P<WS>\s+)"
    LE = r"(?P<LE><=)"  # 匹配优先度
    GE = r"(?P<GE>>=)"
    LT = r"(?P<LT><)"
    GT = r"(?P<GT>>)"
    LBRA = r"(?P<LBRA>\()"
    RBRA = r"(?P<RBRA>\))"
    import re
    from collections import namedtuple
    pat = re.compile("|".join([NAME, VAL, EQ, TIME, PLUS, WS, LT, GT, LBRA, RBRA]))
    def gen_token(text):
        sc = pat.scanner(text)
        Token = namedtuple("Token", ["type", "value"])
        for m in iter(sc.match, None):
            if m.lastgroup != "WS":
                yield Token(m.lastgroup, m.group())

    class Parser(object):
        def __init__(self, text):
            self.expr = text
            self.gen = gen_token(text)
            self.cur_tok = None
            self.next_tok = None

            self._advance()

        def _advance(self):
            self.next_tok, self.cur_tok = next(self.gen, None), self.next_tok

        def _accept(self, tok_type):
            if self.next_tok and self.next_tok.type == tok_type:
                self._advance()
                return True
            return False

        def _require(self, tok_type):
            if not self._accept(tok_type):
                raise SyntaxError("Expect syntax" + tok_type)

        def cal(self):
            return self.cal_plus_or_minus()

        def cal_print(self):
            return print("{} = {}".format(self.expr, self.cal()))

        def cal_plus_or_minus(self):
            # 加减只负责衔接乘除
            ret = self.cal_mul_or_div()
            while self._accept("PLUS") or self._accept("MINU"):
                ret += self.cal_mul_or_div()
            return ret

        def cal_mul_or_div(self):
            # 乘除贪婪计算
            # 每次成功都会处理两个字符第一个一个符号第二个num，自动衔接
            ret = self.evaluate()
            while self._accept("TIME") or self._accept("DIV"):
                cur = self.cur_tok
                next_val = self.evaluate()
                if cur.type == "TIME":
                    ret *= next_val
                elif cur.type == "DIV":
                    ret /= next_val
            return ret

        def evaluate(self):
            if self._accept("VALUE"):
                return int(self.cur_tok.value)
            return 0

    parser = Parser("3 * 4 + 1 * 1")
    parser.cal_print()

    """"""
@splite_out
def practice_2_17():
    #  字节字符串, bytes, bytearray, str
    # str和byte的转换就是encdoe/decode
    # str存储的是unicode(有一个字符映射规则的的全集key)
    # bytes存储0-255(映射ascii，unicode包含ascii的映射)
    # bytearray是可变bytes
    bs = b'Hello wrold'
    bas = bytearray(b"Hello world")
    bas2 = bytearray("Hello world我", encoding="utf8")
    ss = "hello world"
    print(bs, bs[0], type(bs), type(bs[0]))
    print(bas, bas[0], type(bas), type(bas[0]))
    print(bas2, bas2[0], type(bas2), type(bas2[0]))
    print(ss, ss[0], type(ss), type(ss[0]), end="\n\n")

    print(bs[:10], bs.split(), bs.startswith(b"Hello"))
    print(bs.replace(b"wrold", b"world"))
    import re
    print(re.split(bs, b"Hello"), re.sub(b"wrold", b"world", bs), end="\n\n")
    
    # print(b"{} {}".format(b"hello", b"world"))  # 无格式化
    print("{} {}".format("hello", "world").encode("ascii"))
    print("{} {}".format("hello", "world").encode("utf8"), end="\n\n")

    with open("p_2_17.txt", "w") as f:
        f.write("test\n")
    import os
    print(os.listdir("."))
    print(os.listdir(b"."))

if __name__ == "__main__":
    practice_2_1()
    practice_2_2()
    practice_2_3()
    practice_2_4()
    practice_2_5()
    practice_2_6()
    practice_2_7()
    practice_2_8()
    practice_2_9()
    practice_2_10()
    practice_2_11()
    practice_2_12()
    practice_2_13()
    practice_2_14()
    practice_2_15()
    practice_2_16()
    practice_2_17()
```
