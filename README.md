# working_project
> [!NOTE]
> THIS REPOSITORY IS JUST A REPO SHOWING SOME FEW SELECTED PARTS OF THE PROJECT, AND DOES NOT SHOW THE FULL PROJECT.
 
> [!IMPORTANT]
> THIS PROJECT IS BOUND TO SHELL AND ONLY WORKS ON A TERMINAL,FURTHER UPGRADE OF THE PROJECT HAS BEEN INCORPORATED INTO A TELEGRAM BOT AND ALSO BEEN INTEGRATED INTO TKinter for GUI

### ***Python project to store workers and their individual information and also to control inventory database, logging of changes, logging of information of worker that made specific changes***

### ***This project involves the integration of sqlite3 for handling sql commands and Pandas for handling the .CSV databases***

    The databases format used in this project were
       SQL
       .CSV file
       


 
**The database control of wokers and their information using Python + SQL**
> code containing different functions controling the adding, removing, generating unique ids, updatinng worker info...
```python
    database = "admin_data.db"
    con = sql.connect(database)
    cur = con.cursor()
```
> adding worker

```python
    def add_workers():
    try:
        if level_check(): #if it's true
            checks = True
            while checks:
                try:
                    # to make sure the unique_id is not used already as a unique_id or password since selecting the whole list of unique id gives a particular format we dont want
                    to_insert = input("\n\twrite info in format of lastname,firstname, password, email, admin(yes or no), \n\tlevel(manager=1, assist manager=2, IT= 3, inventory managers=4, cashiers =5, others = 6......SEPARATED BY COMMA[,])\n")
                    inserting = r.sub(r"[ ]", "", to_insert)
                    insertions = inserting.split(",", 6)
                    print(insertions)
                    lastname = insertions[0].lower()
                    firstname= insertions[1].lower()
                    password = insertions[2].lower()
                    uid = unique_id()
                    mail = insertions[3]
                    administrator = insertions[4]
                    if administrator.lower() == "yes":
                        admin = True
                    else:
                        admin = False
                    level = insertions[5]
                    #checking easily if unique id is same as already existing unique id or password but checking list indexing True instead of using the list elements
                    uids = cur.execute(f"select unique_id from  users where unique_id = {uid} ")## we have made uid in a way that it cant repeat itself from the generation so we are still safe with uid
                    check_uid = uids.fetchall()
                    names_passes = cur.execute(f"select password from users where password = '{password}'")
                    name_pass = names_passes.fetchall()
                    if not check_uid and not name_pass:
                        try:
                            cur.execute(f"insert into users (last_name, first_name, password, unique_id, email, is_admin, level) values ('{lastname}', '{firstname}', '{password}', {uid}, '{mail}', {admin}, {level})")
                            #con.commit()
                            print("successfully added\n")
                            id_check = cur.execute(f"select unique_id from users where first_name='{firstname}' and  last_name ='{lastname}'and password='{password}' ")
                            id_show = id_check.fetchall()
                            print(f"\n\t\t New user {firstname} {lastname} \b UNIQUE ID IS {id_show[0][0]} ")
                        except Exception as e:
                            print("not added, Error:", e)
                except Exception as e:
                    print("invalid format...Error: ",e )

    except Exception as e:
        print(e)
    except KeyboardInterrupt as e:
        print("")

```
> updatinng worker information
 
```python
    def update_info():##using unique id to get worker info accurately
    try:
        if level_check():
            worker_id = input(f"\n\tenter unique id of worker to change[6 digits]:  ")
            wid= cur.execute(f"select last_name from users where unique_id = {worker_id}")
            uid = wid.fetchall()
            def user_retry():
                if uid:
                    try:
                        change = int(input("\n\tchange how many info?? [1-6][lastname, Firstname, Password, Email, Admin, level]  "))
                        if change <= 6:
                            change_list = []
                            changing = input(f"\n\tchange lastName=1, FirstName= 2, Password=3, Email = 4, admin[yes or no]= 5, level[1-6] = 6 accordingly  ")
                            change_dict = {0:"last_name", 1:"first_name", 2:"password", 3:"Email", 4:"admin", 5:"level"}
                            changes_clean= r.sub(r"[,\s]+", "", changing)
                            if len(changes_clean) == change:
                                for each in changes_clean:
                                    changes = int(each)
                                    changes_list = changes-1
                                    change_list.append(changes_list)
                                full_list = change_list
                                val_list = []
                                info_list =[]
                                print(full_list)
                                # if change == 1:
                                #     cur.execute("update ")
                                identification = cur.execute(f"select first_name, last_name from users where unique_id = {worker_id}")
                                identify = identification.fetchall()
                                firstname= identify[0][0]
                                lastname = identify[0][1]
                                for i in full_list:
                                    key = i
                                    if key in change_dict:
                                        value = change_dict[key] ##just figured out this is where execute many comes in from python-sql but won't use it (:
                                        info_list.append(value)
                                        values_list = input(f"\n\tenter new info of {value} for {firstname} {lastname}: ")
                                        val_list.append(values_list)
                                for x,y in zip(info_list, val_list):
                                    if x == "admin":
                                        if y == "yes":
                                            cur.execute(f"update users set {x} = True where unique_id = {worker_id}")
                                            #con.commit()
                                        else:
                                            cur.execute(f"update users set {x} = False where unique_id = {worker_id}")
                                            #con.commit()
                                    elif x == "level":
                                        z = int(y)
                                        if z > 6:
                                            print("enter value between 1 and 6")
                                            user_retry()
                                        else:
                                            cur.execute(f"update users set {x} = {y} where unique_id = {worker_id}")
                                            #con.commit()
                                    else:
                                        cur.execute(f"update users set {x} = {y} where unique_id = {worker_id}")
                                        #con.commit()
                            else:
                                print("\n\tnumber of information to change doesn't match info inserted ")
                                user_retry()

                        elif change > 6:
                            print("\n\tinfo to change can't be more than 6")
                            user_retry()


                    except ValueError:
                        print("\n\tinput a number not greater than 6 and NOT a letter")
                        user_retry()

                else :
                    print("\n\tWorker doesn't exist, try again ")
                    update_info()

            user_retry()

    except KeyboardInterrupt:
        print("\n\nCANCELLED BY USER")

```

**The database control of inventory items and their information using Python + .CSV database with Pandas**
> To confirm worker and check worker level
```python
    def worker():
    try :
        global retries
        retries = True
        while retries:
            try:

                name = input("enter your full name here [firstname lastname] format: ")
                name_sub = r.sub(r"[,\s]+", " ", name)
                name_split = r.split(r"[\s]", name_sub)
                first = name_split[0]
                last = name_split[1]
                full_name = cur.execute(f"select first_name, last_name from users where last_name = '{last}' and first_name = '{first}'")
                name_list = full_name.fetchall()
                if name_list:
                    user_password = input("\n\tenter your login password: ")
                    user_uid = int(input("\n\tenter your worker id: "))
                    #name_check = r.split(r"[\s,]+", name, )

                    names = cur.execute(f"select password, level, unique_id from users where last_name= '{last}' and first_name = '{first}' and unique_id = {user_uid}  ")
                    names_res = names.fetchall()
                    if not names_res:
                        retrying = input("not a worker..retry? [y or n]")
                        if retrying.lower() == "y":
                            worker()
                        elif retrying == "n":
                            print("exiting ")
                            t.sleep(1)
                        else:
                            print("invalid entry, exiting")
                            t.sleep(1)
                    else:
                        try:
                            password = names_res[0][0]
                            level = names_res[0][1]
                            uid = int(names_res[0][2])
                            if user_password == password and level <= 4 :
                                retries = False
                                return True
                            elif user_password == password and level > 4:
                                retries = False
                                print("cashiers and lower level workers can't access inventory ")
                                return False
                            else:
                                redo = input('invalid  credentials, retry (y or n)? ')
                                t.sleep(1)
                                if redo.lower() == "y":
                                    worker()
                                elif redo.lower() == "n":
                                    t.sleep(1)
                                    print("exiting")
                                else:
                                    print("invalid entry")
                                    t.sleep(1)
                                    print('exiting')
                        except ValueError:
                            print("\n\tworker id must be a 6 digit number only")
                            t.sleep(2)
                            worker()
                else:
                    print(f"Worker {first} {last} does not exist")
                    worker()
            except IndexError:
                print("follow full name [firstname lastname] format ")
                worker()

    except sql.OperationalError:
        print("sql error")
    except ValueError:
        print("\n\tworker id must be a 6 digit number only")
        t.sleep(2)
        worker()
```

> updating values of inventory items
```python
    def change_val():
    try:
        if worker():
            change = pd.read_csv("inventory.csv", index_col="id")
            show_inventory_csv()
            def changing():
                t.sleep(0.4)
                try:

                    #going to check if entered columns are available
                    check_columns = change.columns
                    item = input("\n\tenter the item column to change: ")
                    if item in check_columns:
                        def proceed1():

                            item_val = input("\n\tenter item name to change: ")
                            select = (change[f"{item}"]== f"{item_val}").any()
                            if not select:
                                print(f"\n\tItem {item_val.upper()} does not exist in column {item.upper()}")
                                proceed1()
                            else:
                                def proceed2():

                                    value_col = input("\n\tenter column name containing the value[fruits  fruit_val   auto  auto_val kicthen_utils...]: ")
                                    if value_col in check_columns:
                                        new_value = input("\n\t enter new value: ")
                                        #change.loc[change[f"{item}"]].replace({first_val:new_value})
                                        change.loc[change[f"{item}"]==f"{item_val}", f"{value_col}" ] = float(new_value)
                                        select = change.loc[change[f"{item}"]==f"{item_val}"]
                                        print("\n\t\t\tnew values::\n ")
                                        print(select,"\n")
                                        print("\n\t\t---------------\t\t--------------------------------\t\t----------------")
                                        change.to_csv("inventory.csv", index= True)
                                        reading = pd.read_csv("inventory.csv", index_col="id")
                                        print(reading)
                                        #show_inventory_csv()
                                    else:
                                        print(f"\n\tenter correct item column, {value_col} column does not exist")
                                        proceed2()

                                proceed2()

                        proceed1()
                    else:
                        print(f"\n\tenter correct item column, {item} column does not exist")
                        changing()
                except KeyError:
                    print("enter correct values")
                    changing()
                except ValueError:
                    print("\n\tERROR : new value should be a number")
                    changing()

            changing()
    except Exception as e:
        print("error:  ", e)
    except KeyboardInterrupt:
        print("\n\nCANCELLED BY USER")
```
