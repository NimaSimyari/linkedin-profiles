
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import time
import sqlite3
import pandas as pd

# Your original path. You have to change this pasth from your local pc address
conn_original = sqlite3.connect('C:\\Users\\Rayanegostar\\Desktop\\partydatayork\\Data_spilit3\\group_34.db')

query = 'SELECT * FROM YORK_DATA_FROM_RG_WITH_PROFILE'
df_original = pd.read_sql_query(query, conn_original)

# Close data base
conn_original.close()

# your path. you have to change it. You have to change this pasth from your local pc address
conn_new = sqlite3.connect('C:\\Users\\Rayanegostar\\Desktop\\partydatayork\\linkedin_data_from_rg1\\linkedin_group_34.db')

df_new = df_original.copy()  # Create new copy

# Create new columns
df_new['CON_INFO'] = None
df_new['PRE_UNI'] = None
df_new['LANGUAGE'] = None
df_new['PROFILE'] = None

linkedin_username = 'linkedin_username' # Enter your username
linkedin_password = 'linkedin_password' # Enter your Password

driver = webdriver.Chrome()
driver.get('https://www.linkedin.com')


username_input = driver.find_element('name', 'session_key')
password_input = driver.find_element('name', 'session_password')

username_input.send_keys(linkedin_username)
password_input.send_keys(linkedin_password)
password_input.send_keys(Keys.RETURN)
# In order to avoid captch error I wrote this line.
# If you faced with captch error on Linkedin (solve puzzle) first you should solve it manually. Do not panik you have enough time and code will not be interrupted. After that you enter something in user input and press enter button.
user_input = input("Enter something to if you are human ")

time.sleep(10)



for index, row in df_new.iterrows():
    search_query = f'{row["column1"]} york university' # In my data base I have a column with the name of coulumn1.  In this column there are names of people which I scraped from google scholar or research gate, so I add the york university at the of each name to find the better result.
    search_box = driver.find_element(By.CLASS_NAME, 'search-global-typeahead__input')
    search_box.clear()
    search_box.send_keys(search_query)
    search_box.send_keys(Keys.RETURN)

    try:
        WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.XPATH, f'//span[contains(text(), "{row["column1"]}")]'))
        ).click()

        WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.CLASS_NAME, 'search-global-typeahead__input'))
        )

        # keyword check
        check_keywords = ['york university', 'toronto', 'canada', 'on', 'ontario']
        found_keywords = False
        for keyword in check_keywords:
            if driver.find_elements(By.XPATH, f'//*[contains(text(), "{keyword}")]'):
                found_keywords = True
                break

        if found_keywords:
            # Profile
            contact_info_button = WebDriverWait(driver, 10).until(
                EC.presence_of_element_located((By.XPATH, '//*[@id="profile-content"]/div/div[2]/div/div/main/section[1]/div[2]/div[2]/div[2]/span[2]'))
            )
            contact_info_button.click()
            
            try:
                profile_info_element = WebDriverWait(driver, 10).until(
                    EC.presence_of_element_located((By.XPATH, '//*[@id="profile-content"]/div/div[2]/div/div/main/section[1]/div[2]/div[2]/div[1]/div[2]'))
                )
                profile_info = profile_info_element.text
                df_new.at[index, 'PROFILE'] = profile_info
                
            except Exception as e:
                pass
                    
                
                
                

            try:
                # Contact_info
                contact_info_element = WebDriverWait(driver, 20).until(
                    EC.presence_of_element_located((By.XPATH, '/html/body/div[3]/div/div/div[2]/section/div/section[1]/div'))
                )
                contact_info_text = contact_info_element.text
                df_new.at[index, 'CON_INFO'] = contact_info_text
            except Exception as e:
                pass
            finally:
                driver.back()

            try:
                # Education
                education_button = WebDriverWait(driver, 10).until(
                    EC.presence_of_element_located((By.ID, 'navigation-index-see-all-education'))
                )
                education_button.click()

                education_section = WebDriverWait(driver, 10).until(
                    EC.presence_of_element_located((By.CLASS_NAME, 'pvs-list__container'))
                )
                education_info = education_section.text
                df_new.at[index, 'PRE_UNI'] = education_info
            except Exception as e:
                education_section = WebDriverWait(driver, 10).until(
                    EC.presence_of_element_located((By.XPATH, '//*[@id="profile-content"]/div/div[2]/div/div/main/section[6]/div[3]'))
                )
                education_info = education_section.text
                df_new.at[index, 'PRE_UNI'] = education_info
                pass
            finally:
                driver.back()

            try:
                # Language
                languages_button = WebDriverWait(driver, 10).until(
                    EC.presence_of_element_located((By.ID, 'navigation-index-see-all-languages'))
                )
                languages_button.click()

                languages_section = WebDriverWait(driver, 10).until(
                    EC.presence_of_element_located((By.CLASS_NAME, 'pvs-list__container'))
                )
                languages_info = languages_section.text
                df_new.at[index, 'LANGUAGE'] = languages_info
            except Exception as e:
                languages_section = WebDriverWait(driver, 10).until(
                    EC.presence_of_element_located((By.XPATH, '//*[@id="profile-content"]/div/div[2]/div/div/main/section[6]/div[3]'))
                )
                languages_info = languages_section.text
                df_new.at[index, 'LANGUAGE'] = languages_info
                pass
            finally:
                driver.back()

            
            

    except Exception as e:
        pass

# Save data
df_new.to_sql('LINKEDIN_GROUP1_FROM_RG', conn_new, index=False, if_exists='replace')
conn_new.close()
driver.quit()
