import requests
import schedule
import time
from datetime import datetime
base_cowin_url = "https://cdn-api.co-vin.in/api/v2/appointment/sessions/public/calendarByDistrict"
now = datetime.now()
today_date = now.strftime("%d-%m-%Y")
api_url_telegram = "https://api.telegram.org/bot(This will be telegram bot api)/sendMessage?chat_id=@__groupid__&text="
group_id = "Cowin_Bot_UEM"

WestBengal_district_ids = [710,711,712,713,714,716,718,719,720,721,722,723,724,725,726,727,7283,729,730,731,732,733,734,735,736,737]
def do_nothing():
    def fetch_data_from_cowin(district_id):
        query_params = "?district_id={}&date={}".format(district_id, today_date)
        headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:89.0) Gecko/20100101 Firefox/89.0"}

        final_url = base_cowin_url+query_params
        response = requests.get(final_url,headers=headers)
        extract_availability_data(response)
        #print(response.text)

    def fetch_data_for_state(district_ids):
        for district_id in district_ids:
            fetch_data_from_cowin(district_id)

    def extract_availability_data(response):
        response_json = response.json()
        for center in response_json["centers"]:
            for sessions in center["sessions"]:
                if sessions["available_capacity_dose1"]>1 :
                    message = "Pincode: {}, Name: {}, Available_Does: {}, Minimum Age: {}".format(
                          center["pincode"], 
                          center["name"], 
                          sessions["available_capacity_dose1"], 
                          sessions["min_age_limit"])
                    send_message_telegram(message)



    def send_message_telegram(message):
        final_telegram_url = api_url_telegram.replace("__groupid__",group_id)
        final_telegram_url = final_telegram_url + message
        response= requests.get(final_telegram_url)
        print(response)






    if __name__ == "__main__":
            fetch_data_for_state(WestBengal_district_ids)
            
            
schedule.every(300).seconds.do(do_nothing) 
while 1:
    schedule.run_pending()
    time.sleep(1)
