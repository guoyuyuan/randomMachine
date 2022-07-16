from PySide2.QtWidgets import QApplication, QMainWindow, QMessageBox, QWidget
from PySide2.QtUiTools import QUiLoader
from PySide2.QtGui import  QIcon
import random

class Main:
    def __init__(self):
        # 加载ui文件
        self.ui = QUiLoader().load('ui/random_machine.ui')

        # 创建一个空列表，用于存储生成的指定数字
        self.number_list = []

        # 设置次数
        self.count = 0

        # 历史记录 仅限用于测试输出打印，并没用实际的作用效果
        self.history = []

        # 设置标识符，用来判断重置按钮是否被点击  False表示默认没有点击
        self.reset_clicked = False

        # 创建一个空字符串，用于存储显示到界面的历史记录信息
        self.information = "*"*50 + "重新启动了程序" + "*"*50


        # 执行主程序，即自动调用各种方法
        self.action()


    # 设计一个方法，用于自动调用各种方法
    def action(self):
        # 每重启程序一次就往日志文件加一行分割符
        self.write_text(self.information)
        # 获取起始数字
        self.first_number = self.first_textEdit()
        # 获取结束数字
        self.last_number = self.last_textEdit()
        # 调用方法，生成指定的数组
        self.get_number_list()
        # 按钮点击事件
        self.button_action()


    # 按钮点击事件
    def button_action(self):
        # 开始按钮
        self.ui.start_pushButton.clicked.connect(self.random_machine)
        # 重置按钮
        self.ui.reset_pushButton.clicked.connect(self.reset_button)
        # 清空历史记录按钮
        self.ui.clear_history_pushButton.clicked.connect(self.clear_history_pushButton)
        # 默认按钮
        self.ui.default_pushButton.clicked.connect(self.default_pushButton)


    # 抽取数字
    def random_machine(self):
        print(f"当前的重置按钮状态：{self.reset_clicked}")

        # 一开始就判断起始数字和结束数字是否被修改 如果修改就将状态改为False
        # 因为如果不调用isNumber_modify()的话，那么一开始就要点击重置按钮 这样用户体验感就不是很好
        self.isNumber_modify()
        # 获取到最新的开始数字
        first_number = self.first_textEdit()
        # 获取最新的结束数字
        last_number = self.last_textEdit()

        # 判断用户给的区间是否合理，即是否起始数字小于结束数字
        if first_number > last_number:
            QMessageBox.about(self.ui,
                              "温馨提示",
                              f"请保证起始数字小于结束数字",
                              )
            # 记录 历史记录
            self.information = "起始数字大于结束数字"
            self.history.append(self.information)
        else:
            if self.reset_clicked == False:
                # 如果起始数字和结束数字被修改了，那就将开始按钮禁用和提示用户点击重置按钮
                if self.first_number!=first_number or self.last_number!= last_number:
                    QMessageBox.about(self.ui,
                                      "温馨提示",
                                      f"源源发现你修改了起始或者结束数字，请先点击重置按钮",
                                      )
                    # 将开始按钮禁用
                    self.ui.start_pushButton.setEnabled(False)

                    # 记录 历史记录
                    self.information = "你修改了起始或者结束数字"
                    self.history.append(self.information)

            else:
                # 根据用户指定的次数 进行循环
                self.count += 1
                # 如果原始数据还有数据
                if len(self.number_list) != 0:
                    # 调用 删除已经随机生成的数字 方法
                    number = self.remote_number(self.count)
                    self.show_number(number)
                # 判断当前次数是否大于结束数字，如果大于就说明数据已经抽取完毕了
                if self.count >= last_number+1:
                    QMessageBox.about(self.ui,
                                      "温馨提示",
                                      f"本次抽取的数字已完成，请点击重置按钮和重新设置数字重新开始",
                                      )
                    # 将开始按钮禁用
                    self.ui.start_pushButton.setEnabled(False)
                    # 记录 历史记录
                    self.information = "本次抽取的数字已完成"
                    self.history.append(self.information)


                # 显示历史记录
                self.show_history()

                # 将历史记录写入日志文件
                self.write_text()

                print(self.history)

    # 设计一个方法，用于生成指定的数据
    def get_number_list(self):
        # 获取起始数字
        first_number = self.first_textEdit()
        # 获取结束数字
        last_number = self.last_textEdit()

        # 将count值设为0，重新开始生成新数据
        self.count = 0
        for i in range(first_number, last_number + 1):
            self.number_list.append(i)
        print(f"开始的数据：{self.number_list}")
        print("*" * 100)

    # 设计一个方法，用于删除已经随机生成的数字
    def remote_number(self, count):
        # 根据已有的列表 随机生成数
        number = random.choice(self.number_list)
        # 根据随机生成的数 从 原始数据中 删除
        self.number_list.remove(number)

        self.information = f"第{count}次随机生成的数是:{number}"
        self.history.append(self.information)
        print(f"第【{count}】次删除了:{number}")
        print(f"第【{count}】次删除后的数据:{self.number_list}")
        # 判断原始数据中是否还包含删除过的数据
        is_contain = number in self.number_list
        print(f"(number_list是否包含{number}:{is_contain})")
        return number



    # 获取起始数字
    def first_textEdit(self):
        value = self.ui.first_textEdit.toPlainText()
        # 判断用户是否给了空字符串
        self.is_user_input_Empty(value)
        if " " or "\n" in value:
            value = self.ui.first_textEdit.toPlainText().replace(" ", "").replace("\n", "")
        print(f"起始数字：{value}")
        return int(value)

    # 获取结束数字
    def last_textEdit(self):
        value = self.ui.last_textEdit.toPlainText()
        # 判断用户是否给了空字符串
        self.is_user_input_Empty(value)
        # 判断用户给的字符串是否有空格或者换行
        if " " or "\n" in value:
            value = self.ui.last_textEdit.toPlainText().replace(" ", "").replace("\n", "")
        print(f"结束数字：{value}")
        return int(value)

    # 展示生成的随机数
    def show_number(self,number):
        self.ui.main_textEdit.setText(str(number))


    # 显示历史记录
    def show_history(self):
        self.ui.history_plainTextEdit.appendPlainText(str(self.information)+"\n")


    # 重置按钮
    def reset_button(self):
        # 当重置按钮被点击了，就将flag设置为True
        self.reset_clicked = True
        first_number = self.first_textEdit()
        last_number = self.last_textEdit()
        self.first_number = first_number
        self.last_number = last_number
        print("*"*100)
        print("你点击了重置按钮")
        # 记录 历史记录
        self.information = f"===你点击了重置按钮==="
        self.history.append(self.information)
        # 将开始按钮启用
        self.ui.start_pushButton.setEnabled(True)
        # 重置原来的数据
        self.number_list.clear()
        # 重新生成数据
        self.get_number_list()
        # 设置主文本框内容为空
        self.ui.main_textEdit.setText('')

    # 清空历史记录按钮
    def clear_history_pushButton(self):
        self.ui.history_plainTextEdit.clear()

        # 记录 历史记录 信息
        self.information = f"清空了历史记录"
        self.history.append(self.information)

    # 将起始数字和结束数字恢复默认值
    def default_pushButton(self):
        # 默认初始值为0 和 50
        self.ui.first_textEdit.setText('0')
        self.ui.last_textEdit.setText('50')

        # 禁用开始按钮
        self.ui.start_pushButton.setEnabled(False)
        # 弹出提示框提示用户点击重置按钮
        QMessageBox.about(self.ui,
                          "温馨提示",
                          "你点击默认按钮了，请先点击重置按钮\n,才可以点击开始按钮")

        # 记录 历史记录 信息
        self.information = f"点击了默认按钮"
        self.history.append(self.information)


    # 如果修改了起始和结束数据，那就将原始的默认数据重置
    def reset_default_number_list(self):
        # 启用开始按钮
        self.ui.start_pushButton.setEnabled(True)
        # 重置原来的数据
        self.number_list.clear()
        # 重新生成数据
        self.get_number_list()

    # 判断起始数字和结束数字是否被修改了
    def isNumber_modify(self):
        first_number = self.first_textEdit()
        # 获取最新的结束数字
        last_number = self.last_textEdit()

        # 如果起始数字和结束数字被修改了，那就修改重置按钮状态
        if self.first_number != first_number or self.last_number != last_number:
            self.reset_clicked = False

        else:
            self.reset_clicked = True
        # print(self.first_number,self.last_number)
        # print(first_number,last_number,self.reset_clicked)

    # 设计一个方法，判断用户给的数字是否为空
    def is_user_input_Empty(self,value):
        # if self.first_number == "" or self.last_number == "":
        if value == "":
            QMessageBox.about(self.ui,
                              "温馨提示",
                              f"你还没有设置起始和结束数字嘞",
                              )
            # 记录 历史记录
            self.information = "起始或结束数字为空"
            self.history.append(self.information)
            print("用户给了个空字符串")
            return False
        else:
            return True



    # 将历史记录写入日志文件中
    def write_text(self,information=None):
        # （将information默认值设为None 如果information的内容为None说明重启了程序，
        # 即添加分割线 用于区分每次的启用程序）
        information = self.information
        import time
        # 获取当前的时间
        now_time = time.strftime('%Y-%m-%d %H:%M:%S',time.localtime())
        # 将历史记录写入日志文件中
        with open('txt/log.txt', 'a') as f:
            if information != None:
                f.write(now_time + "：" + self.information + "\n")
            else:
                f.write(now_time + "：" + information + "\n")
        print("file Written Successfully")

if __name__ == '__main__':
    import ctypes
    # 加载ui文件
    app = QApplication([])
    # 加载 icon
    app.setWindowIcon(QIcon('imgs/random_logo.png'))
    # 将任务栏的图标跟窗口上的图标设为一致
    ctypes.windll.shell32.SetCurrentProcessExplicitAppUserModelID("myappid")
    m = Main()
    m.ui.show()
    app.exec_()
    # 在cmd中执行以下命令。即可将该程序打包成exe文件
    # pyinstaller main.py --noconsole --hidden-import PySide2.QtXml --icon="imgs/random_logo.ico"

