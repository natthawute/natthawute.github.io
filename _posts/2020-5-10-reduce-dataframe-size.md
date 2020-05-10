---
layout: post
title: "วิธีลดขนาดของ dataframe ใน python"
tags:
    - performance
    - tips
--- 
วันนี้เราจะมาดูวิธีลดขนาดของ dataframe ตอนที่ทำงานบน dataset ใหญ่ ๆ เพื่อประหยัด
memory กันครับ
โดยวิธีที่แนะนำวันนี้ผมไม่ได้คิดเองแต่อย่างใด แต่เห็นใช้กันอย่างแพร่หลายใน
kaggle ครับ 
 
เริ่มจาก import library ต่าง ๆ แหละ โหลด data ของเรามาก่อนครับ 


<div class="input_area">
<div class="input_prompt">
In [1]:
</div>
{% highlight python %}
import pandas as pd
import numpy as np
{% endhighlight %}
</div>


<div class="input_area">
<div class="input_prompt">
In [2]:
</div>
{% highlight python %}
df = pd.read_csv('dummy_data.csv')
{% endhighlight %}
</div>
 
จากนั้นเขียนฟังก์ชันเพื่อทำการลดขนาด dataframe โดยหลักการคร่าว ๆ คือปกติเนี่ย
default type ของ int และ float จะเป็น int64 และ float64 ครับ
ซึ่งมันสามารถเก็บค่าได้เยอะมาก
แต่หลาย ๆ ครั้ง เราไม่ได้ต้องการเก็บค่าเยอะขนาดนั้น เราเลยสามาถลดขนาดของ column
พวกนี้ให้ต่ำลงมาเป็น int16, int32 หรือ float16, float32 ได้ครับ
ซึ่งก็จะทำให้ dataframe เราลดขนาดลงไปได้พอสมควรเลย 


<div class="input_area">
<div class="input_prompt">
In [3]:
</div>
{% highlight python %}
# modified from https://www.kaggle.com/ragnar123/very-fst-model
def reduce_mem_usage(df):
    numerics = ['int16', 'int32', 'int64', 'float16', 'float32', 'float64']
    start_mem = df.memory_usage().sum() / 1024**2    
    for col in df.columns:
        col_type = df[col].dtypes
        if col_type in numerics:
            c_min = df[col].min()
            c_max = df[col].max()
            if str(col_type)[:3] == 'int':
                if c_min > np.iinfo(np.int8).min and c_max < np.iinfo(np.int8).max:
                    df[col] = df[col].astype(np.int8)
                elif c_min > np.iinfo(np.int16).min and c_max < np.iinfo(np.int16).max:
                    df[col] = df[col].astype(np.int16)
                elif c_min > np.iinfo(np.int32).min and c_max < np.iinfo(np.int32).max:
                    df[col] = df[col].astype(np.int32)
                elif c_min > np.iinfo(np.int64).min and c_max < np.iinfo(np.int64).max:
                    df[col] = df[col].astype(np.int64)  
            else:
                if c_min > np.finfo(np.float16).min and c_max < np.finfo(np.float16).max:
                    df[col] = df[col].astype(np.float16) #float16 can't save to feather
                    df[col] = df[col].astype(np.float32)
                elif c_min > np.finfo(np.float32).min and c_max < np.finfo(np.float32).max:
                    df[col] = df[col].astype(np.float32)
                else:
                    df[col] = df[col].astype(np.float64)    
    end_mem = df.memory_usage().sum() / 1024**2
    print('Mem. usage decreased from {:5.2f} Mb to {:5.2f} Mb ({:.1f}% reduction)'
          .format(start_mem, end_mem, 100 * (start_mem - end_mem) / start_mem))
    return df
{% endhighlight %}
</div>


<div class="input_area">
<div class="input_prompt">
In [4]:
</div>
{% highlight python %}
df = reduce_mem_usage(df)
{% endhighlight %}
</div>

    Mem. usage decreased from 153.96 Mb to 107.60 Mb (30.1% reduction)

 
จะเห็นได้ว่า dataframe ของเราลดขนาดของ ประมาณ 153.96 Mb เหลือแค่ 107.6 Mb
เท่านั้นครับ
เท่ากับว่าขนาดลงลงไปถึง 30% เลยทีเดียว 
