# get-ticket

    # -*- coding: utf-8 -*-

    """
    @author:xiaoMaGe
    """

    'it's rob ticket for myself as begin. '

    from selenium import webdriver
    from selenium.common import exceptions
    import random
    from selenium.webdriver.common.keys import Keys
    from bs4 import BeautifulSoup
    import time
    import datetime


    class get_ticket(object):
        """docstring for get_ticket"""
        # 用户名,密码
        username = ""
        passwd = ""
        # 出发地,目的地
        start_way = ""
        end_way = ""
        # 乘车人
        users = []
        # 出发日期
        fromDate = ""
        # 列车类型 车次 第一个字母
        ticket_type = ""
        # 座位类型
        seat_type = ""
        # 发车时间
        start_time = ""
        # 到达时间
        end_time = ""
        # 刷新间隔时间
        refresh_ts = 1

        """Ticket purchase website"""
        login_url = "https://kyfw.12306.cn/otn/login/init"
        initmy_url = "https://kyfw.12306.cn/otn/index/initMy12306"
        ticket_url = "https://kyfw.12306.cn/otn/leftTicket/init"
        buy_url = "https://kyfw.12306.cn/otn/confirmPassenger/initDc"

        ## 此类的参数
        def __init__(self):
            self.username = self.input_def("请输入12306账号: (如果无,请自行登录时输入)\n")
            self.passwd = self.input_def("请输入12306密码: (如果无,请自行登录时输入)\n")
            self.start_way = self.input_def("请输入出发地: (默认:上海) \n", defult="上海")
            self.end_way = self.input_def("请输入目的地: (默认:长沙) \n", defult="长沙")
            self.fromDate = self.input_def("请输入出发日期: ( 如: 2018-09-23 ) \n", defult="2018-09-23")
            self.start_time = self.input_def("请输入发车时间: (默认为 06:00之后) \n", defult="06:00")
            self.end_time = self.input_def("请输入到达时间: (默认为 24:00之前) \n", defult="24:00")

            self.users = self.input_def("请输入乘车人: (用,分隔 -- '英文输入,号' 必填) \n")
            self.ticket_type = self.input_def("请输入列车类型: ( 如: 'G','D','Z','T','K','QT') 默认高铁 G \n", defult="G")
            self.seat_type = self.input_def("""请输入座位类型: 默认二等座 (\n
                        商务座 :\t 'SWZ'\n
                        一等座 :\t 'ZY'\n
                        二等座 :\t 'ZE'\n
                        高级软卧 :\t 'GR'\n
                        软卧 :\t 'RW'\n
                        动卧 :\t 'SRRB'\n
                        硬卧 :\t 'YW'\n
                        软座 :\t 'RZ'\n
                        硬座 :\t 'YZ'\n
                        无座 :\t 'WZ'\n
                        其他 :\t 'QT'\n) \n""", defult="ZE")

        # 主运行方法
        def start(self):
            try:
                self.driver = webdriver.Firefox()
                # self.driver.find_element_by_xpath("/html/body/div[1]/div/div[2]/ul/li[2]/a").click()
                self.cookie = self.get_cookie()
                # 登录12306
                self.login()
                time.sleep(0.05)
                self.start_ts = time.time()
                while True:
                    try:
                        self.ticket()
                        break
                    except exceptions.ElementNotInteractableException:
                        continue
                # self.driver.close()
            except exceptions as e:
                print(e, e.msg)
        # 查询并购买票
        def ticket(self):
            self.driver.get(self.ticket_url)
            # 添加出发地、目的地、出发日 cookie
            for i in self.cookie:
                self.driver.add_cookie(i)
            self.driver.refresh()
            self.reserve_ticket()
            self.buy_ticket()
        # 输入出发站
        def get_fromStation(self):
            # while True:
                # try:
                    # 在driver中填入出发地
            self.driver.find_element_by_id("fromStationText").clear()
            time.sleep(0.05)
            self.driver.find_element_by_id("fromStationText").click()
            time.sleep(0.05)
            self.driver.find_element_by_id("fromStationText").send_keys(self.start_way)
            time.sleep(0.05)
            self.driver.find_element_by_xpath("//div[@id='citem_0']").click()
                    # break
                # except exceptions.NoSuchElementException as e:
                    # if e.msg == "Unable to locate element: //div[@id='citem_0']":
                        # self.driver.find_element_by_id("fromStationText").click()
                    # continue
        # 输入目的站
        def get_toStation(self):
            # 在driver中填入目的地
            self.driver.find_element_by_id("toStationText").clear()
            time.sleep(0.05)
            self.driver.find_element_by_id("toStationText").click()
            time.sleep(0.05)
            self.driver.find_element_by_id("toStationText").send_keys(self.end_way)
            time.sleep(0.05)
            self.driver.find_element_by_xpath("//div[@id='citem_0']").click()

        # 获取出发地、目的地 cookie
        def get_cookie(self):
            self.driver.get(self.ticket_url)
            self.driver.delete_all_cookies()
            try:
                # 在driver中填入日期cookie
                self.driver.add_cookie({'name': '_jc_save_fromDate', 'value': '{}'.format(self.fromDate)})
                time.sleep(0.05)
                self.driver.refresh()
                time.sleep(0.05)
                self.get_fromStation()
                self.get_toStation()
                time.sleep(0.05)
                self.driver.find_element_by_xpath("//div[@class='btn-area']/a").click()

                cookies = self.driver.get_cookies()
            except exceptions.NoSuchElementException as e:
                print(e, e.msg)
                cookies = [{'name': '_jc_save_fromStation', 'value': '%u4E0A%u6D77%2CSHH'},
                           {'name': '_jc_save_toStation', 'value': '%u957F%u6C99%2CCSQ'},
                           {'name': '_jc_save_fromDate', 'value': self.fromDate}
                           ]
            finally:
                cookie = [i for i in cookies if i['name'] in ["_jc_save_fromDate", "_jc_save_fromStation",
                                                              "_jc_save_toStation"]]
            return cookie
        # 登录账号
        def login(self):
            self.driver.get(self.login_url)
            if self.username != "":
                self.driver.find_element_by_id("username").clear()
                self.driver.find_element_by_id("username").send_keys(self.username)
            if self.passwd != "":
                self.driver.find_element_by_id("password").clear()
                self.driver.find_element_by_id("password").send_keys(self.passwd)
            print("请自行输入验证码...")
            while True:
                if self.driver.current_url != self.initmy_url:
                    time.sleep(1)
                else:
                    break
        # 循环查询票
        def reserve_ticket(self):
            self.driver.find_element_by_xpath("//input[@value='{}']".format(self.ticket_type)).click()
            self.driver.find_element_by_xpath("//*[@id='avail_ticket']").click()
            select_no = 1
            while True:
                # 点击查询
                self.driver.find_element_by_xpath("//div[@class='btn-area']/a").click()
                print("第{}次查询: ".format(select_no))
                time.sleep(0.05)
                for i in self.driver.find_elements_by_xpath("//tbody/tr[starts-with(@id,'ticket_')]"):
                    train_no = i.find_element_by_xpath(".//div[@class='train']").text
                    start_ts = i.find_element_by_xpath(".//strong[@class='start-t']").text
                    end_ts = i.find_element_by_xpath(".//strong[@class='color999']").text
                    seat_u = i.find_element_by_xpath(".//td[starts-with(@id,'{}_')]".format(self.seat_type)).text
                    if start_ts >= self.start_time and end_ts <= self.end_time and seat_u != '无':
                        try:
                            i.find_element_by_xpath("./td[@class='no-br']").click()
                            if self.driver.current_url == self.buy_url:
                                break
                        except exceptions.NoSuchElementException as e:
                            print(e, e.msg)
                            print("{}次列车无选定类型的票...".format(train_no))
                if self.driver.current_url == self.buy_url:
                    break
                select_no += 1
                # 随机时间间隔
                sleep_time = self.refresh_ts + round(random.random(), 2)
                time.sleep(sleep_time)
                # 整个运行时间,
                use_ts = ((time.time() - self.start_ts)//20 + 1) % 60
                if use_ts == 0:
                    self.driver.find_element_by_xpath("//*[@id='regist_out']").click()
                    self.login()
                    self.driver.get(self.ticket_url)
                    for j in self.cookie:
                        self.driver.add_cookie(j)
                    self.driver.find_element_by_xpath("//input[@value='{}']".format(self.ticket_type)).click()
                    self.driver.find_element_by_xpath("//*[@id='avail_ticket']").click()

        # 下订单
        def buy_ticket(self):
            try:
                for user in self.users:
                    self.driver.find_element_by_xpath("//label[text()='{}']/../input".format(user)).click()
                    time.sleep(0.05)
                # 提交订单
                self.driver.find_element_by_id("submitOrder_id").click()
                time.sleep(0.05)
                """
                # 选座
                while self.driver.find_element_by_id("selectNo").text == '0/1':
                    print("请选座...")
                    if self.driver.find_element_by_id("selectNo").text == '1/1':
                        break
                    time.sleep(self.refresh_ts)
                """
                self.driver.find_element_by_id('qr_submit_id').click()
            except exceptions.NoSuchElementException as e:
                print(e)

        def input_def(prin, msg, defult=""):
            print(prin)
            r = input(msg)
            if ',' in r:
                return r.split(",")
            elif r == "":
                return defult
            else:
                return r



    if __name__ == '__main__':
        get_ticket = get_ticket()
        get_ticket.start()

