"""
Таблица orders -я так понимаю здесь ключ поле OrderId.

Таблица orderLines - я так понимаю здесь на каждый OrderId может быть несколько
 ProductId.
 
самые популярные продукты - предполагаю, что здесь нужно просто отсортировать 
по убыванию уникальное количество клиентов на каждый продукт.

суммарную выручку по каждому такому продукту- это понятно.

средний чек заказов, в которых есть такие продукты - я предполагаю, что если 
в заказе продукт повторяется не один раз (а такое может быть, потому что у нас 
нет поля количество продукта), то я этот заказ учитываю столько раз, сколько 
данный продукт в нем находится.

За последний месяц - это понятно.

Для воспроизводимости результата у меня будет "сегодня" всегда 12 июня 2019. 
Надеюсь на понимание. В любом случае это не сложно исправить.
"""

import numpy as np
import pandas as pd
import random


# Функция, которая искусственно создает таблицы orders и orderLines
def create_tables(seed, n_orders, n_uniq_prod, n_cust):
    random.seed(seed)
    # пусть в среднем на каждый заказ будет 4 продукта
    n_pur = n_orders * 4
    # создаем случайно n_orders дат с 1 января по 12 июня, 
    # не включая 29, 30 и 31 числа, для удобства.
    X = [pd.Timestamp(year=2019, month=random.randint(1, 6), 
                day=random.randint(1, 28), hour=12) for i in range(n_orders)]
    today = pd.Timestamp(year=2019, month=6, day=12, hour=0)
    for i in range(len(X)):
        if X[i].date() > today.date():
            X[i] = pd.Timestamp(year=2019, month=random.randint(1, 5), 
             day=random.randint(1, 28), hour=12)

    # создаем случайно n_orders клиентов из диапазона от 1000 до 1000 + n_cust
    costumer = [random.randint(1000, 1000 + n_cust) for i in range(n_orders)]
    Order = [111000 + i for i in range(n_orders)]
    dict_ = {'DateTime': X, 'CustomerId':costumer, 'OrderId':Order}
    orders = pd.DataFrame(dict_)

    # создаем n_uniq_prod продуктов и цен им соответсвующих
    Product = [77000 + i for i in range(n_uniq_prod)]
    Price = [random.randint(1000, 5000) for i in range(n_uniq_prod)]
    numbers = [i for i in range(n_uniq_prod)]

    # создаем случайно n_pur покупок 
    numbers2 = [random.choice(numbers) for i in range(n_pur)]
    Product2 = [Product[numbers2[i]] for i in range(n_pur)]
    Price2 = [Price[numbers2[i]] for i in range(n_pur)]
    Order2 = [random.choice(Order) for i in range(n_pur)]

    dict_ = {'OrderId': Order2, 'ProductId':Product2, 'Price':Price2}
    orderLines = pd.DataFrame(dict_)
    return orders, orderLines
                              
         
# Та самая функция, которая делает отчет, второе возвращаемое значение - отчет.
# Первое возвращаемое значение нужно только для того чтобы другой логикой 
# проверить правильность отчета.                   
def create_report(orders, orderLines):
    # дата, которая месяц назад. Сейчас для воспроизводимости константа.
    start_date = pd.Timestamp(year=2019, month=5, day=12, hour=0)
    df = pd.merge(orderLines, orders.loc[orders['DateTime'] > start_date], 
                  on='OrderId', how='inner')
    df_total = pd.merge(df, df.groupby(by='OrderId', as_index=False).agg(
            {'Price': 'sum'}), on='OrderId', how='inner')
    df_total = df_total.groupby(by='ProductId', as_index=False).agg(
            {'CustomerId': pd.Series.nunique, 'Price_x': 'sum', 
             'Price_y': 'mean'})
    df_total.columns = ['ProductId', 'count_dist_CustomerId', 
                        'sum_of_price', 'AVG_price_order']
    df_total = df_total.sort_values(by=['count_dist_CustomerId'], 
                                    ascending = False)
    return df, df_total

"""
Тесты я решил не смотреть как делают в интернете, а придумал по своему.
Если 2 разных логичных метода выдают один и тот же результат, значит скорее 
всего все правильно. Хотя в будущем я изучу как нужно правильно делать тесты.
у нас есть df - это INNER JOIN orders и orderLines по полю OrderId с учетом 
условия последнего месяца.
df_total - это готовый отчет, сделанный используя pandas.
Теперь проверять мы будем используя прямой метод перебора, работая с таблицей 
df. Так как можно доверять INNER JOIN и условию даты. И так как это прямой 
метод, то он будет работать долго.
"""
def check_report(df, df_total):
    errors = []
    for i in range(df_total.shape[0]):
        prod = df_total['ProductId'].iloc[i]
        count_1 = df_total['count_dist_CustomerId'].iloc[i]
        sum_1 = df_total['sum_of_price'].iloc[i]
        avg_1 = df_total['AVG_price_order'].iloc[i]

        count_2 = len(set(df['CustomerId'][df['ProductId']==prod]))
        sum_2 = sum(df['Price'][df['ProductId']==prod])
        orders_ = list(df['OrderId'][df['ProductId']==prod])
        list_ = []
        for ord in orders_:
            list_.append(sum(df['Price'][df['OrderId']==ord]))
        avg_2 = np.mean(list_)
        err = []
        # проверка на количество уникальных клиентов (популярность)
        if count_1 != count_2:
            err.append([count_1, count_2, 'count'])
         # проверка на общую сумму продукта
        if sum_1 != sum_2:
            err.append([sum_1, sum_2, 'sum'])
         # проверка на среднее по заказам на продукт
        if avg_1 != avg_2:
            err.append([avg_1, avg_2, 'avg'])
         # проверка на одинаковое множество продуктов. Не должно чтобы чего то
         # не хвтатало или было лишним.
        if set(df_total['ProductId']) != set(df['ProductId']):
            err.append(['dif uniq size of ProductId'])
        if err != []:
            errors.append(err)
    return errors


# Мои варианты для тестов
list_count_orders = [1, 100, 1000]
list_count_products = [1, 2, 50, 100]
list_count_customers = [1, 2, 50, 200]
list_seed = [42, 26, 89]

# Прогоняем те варианты, которые я придумал, итого 182 комбинации. Если будет 
# ошибка, то напечатается где именно ошибка и 
# напечатается под ним сразу "_________________ERROR__________________"
# Иначе просто напечатается какая сейчас комбинация отрабатывает.
for my_count_orders in list_count_orders:
    for my_count_products in list_count_products:
        for my_count_customers in list_count_customers:
            for my_seed in list_seed:
                print(my_seed, my_count_orders, my_count_products, 
                      my_count_customers)
                orders, orderLines = create_tables(my_seed, my_count_orders, 
                                        my_count_products, my_count_customers)
                df, df_total = create_report(orders, orderLines)
                errors = check_report(df, df_total)
                if errors != []:
                    print(errors)
                    print('_________________ERROR__________________')
