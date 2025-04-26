---
layout: post
title:  "Vẽ bản đồ dân số bằng Python"
date:   2025-04-25 16:12:00 +0700
tags:   python data choropleth
---

Bản đồ dân số là một loại bản đồ biểu đạt về phân bố dân cư trên một khu vực cụ thể (một xã, thành phố, quốc gia hoặc thế giới).
Bản đồ dân số thường được biểu diễn bằng bản đồ choropleth.

> Bản đồ chorolepth (từ tiếng Hy Lạp χῶρος “khu vực/vùng” và πλῆθος “số lượng”)
> là một loại địa đồ trong đó các khu vực được đổ bóng hoặc thêm hoạ tiết tương
> đương với giá trị của một biến số thống kê. Biến số này đại diện cho một tóm
> tắt tổng hợp về một đặc điểm địa lý trong mỗi khu vực, ví dụ như mật độ dân
> số hay thu nhập trên đầu người.

| ![Bản đồ dân số Thường Tín](/assets/images/thuong-tin.png) |
|:--:|
| *Bản đồ dân số huyện Thường Tín, Hà Nội* |

Muốn vẽ được bản đồ nhiệt ta cần 2 loại dữ liệu:
1. Hình dạng địa lý về của một đơn vị hành chính.
Hình dạng này được biểu diễn bằng kiểu dữ liệu GeoJSON.
Chúng ta sẽ vẽ bản đồ dân số của huyện Thường Tín, thành phố Hà Nội (nơi tui sinh sống).
Dữ liệu được sử dụng trong bài nằm tại [ban-do-thuong-tin.json](/assets/data/ban-do-thuong-tin.json),
bao gồm dữ liệu của các xã tại huyện Thường Tín.
> GeoJSON là một định dạng mở được thiết kế để biểu diễn
> các đặc điểm địa lý đơn giản, cùng với các thuộc tính
> phi không gian của chúng.
2. Dữ liệu cần được biểu diễn, trong bài viết này chúng ta sẽ biểu diễn
mật độ dân số của các xã tại huyện Thường Tín. Dữ liệu này nằm trong
[dan-so-thuong-tin.csv](/assets/data/dan-so-thuong-tin.csv).

Trong bài này ta sẽ sử dụng ngôn ngữ lập trình Python thường được sử dụng
trong phân tích dữ liệu.



# 1. Làm việc với GeoJSON

Đầu tiên, việc bạn cần làm là tải file [ban-do-thuong-tin.json](/assets/data/ban-do-thuong-tin.json).
Bản đồ này là tập hợp dữ liệu của các xã, với dữ liệu của từng xã được biểu diễn như sau:

{% highlight json %}
{
    "id": "1",
    "type": "Feature",
    "properties": {
        "name": "Thị trấn Thường Tín"
    },
    "geometry": {
        "type": "MultiPolygon",
        "coordinates": [[[[105.865905761719,20.8654365539551],[105.857131958008,20.8666362762451],[105.856376647949,20.8688945770265],[105.859237670899,20.868709564209],[105.858299255371,20.8730697631837],[105.860145568848,20.8735637664795],[105.862640380859,20.8742465972901],[105.861068725586,20.8779144287109],[105.861473083496,20.8781509399414],[105.864082336426,20.8758220672607],[105.863594055176,20.8754520416261],[105.864410400391,20.8746185302736],[105.864677429199,20.8723239898682],[105.867362976074,20.8721466064454],[105.866889953613,20.8694934844971],[105.86637878418,20.8681964874269],[105.868949890137,20.8672275543214],[105.868644714355,20.866325378418],[105.867530822754,20.8660488128662],[105.868293762207,20.8651084899902],[105.866439819336,20.8645172119142],[105.865905761719,20.8654365539551]]]]
    }
}
{% endhighlight %}

Dữ liệu của một xã bao gồm tên và địa giới hành chính.
Bây giờ, ta cần đọc file này trong python.

{% highlight python %}
import json

# Đọc file ban-do-thuong-tin.json
with open("ban-do-thuong-tin.json") as f:
    ban_do_thuong_tin = json.load(f)
{% endhighlight %}



# 2. Làm việc với CSV

Bây giờ, bạn hãy tải file [dan-so-thuong-tin.csv](/assets/data/dan-so-thuong-tin.csv) về máy.

{% highlight csv %}
mahanhchinh,ten,dientich,danso
10183,Thị trấn Thường Tín,79,5915 
10186,Xã Ninh Sở,493,7647 
10189,Xã Nhị Khê,281,5582 
10192,Xã Duyên Thái,404,8603 
10195,Xã Khánh Hà,481,8461 
10198,Xã Hòa Bình,392,5196 
10201,Xã Văn Bình,523,8223 
10204,Xã Hiền Giang,321,4110 
10207,Xã Hồng Vân,449,4204
{% endhighlight %}

File này gồm bốn cột dữ liệu bao gồm mã hành chính, tên xã, diện tích (đơn vị 0,01 km2), dân số (năm 2009).
Để đọc được file này ta cần sử dụng thư viện [Pandas](https://pandas.pydata.org/).

{% highlight python %}
import pandas as pd

# Đọc file dan-so-thuong-tin.csv
dan_so = pd.read_csv("dan-so-thuong-tin.csv")
# Vì trong dữ liệu chưa có cột mật độ nên giờ ta cần tính
dan_so["matdo"] = dan_so["danso"] / (dan_so["dientich"]*0.01)
{% endhighlight %}



# 3. Vẽ bản đồ choropleth

Để vẽ được bản đồ choropleth ta cần sử dụng thư viện [Plotly](https://plotly.com/python/).
{% highlight python %}
import plotly.express as px

fig = px.choropleth_map(
    dan_so,
    geojson=ban_do_thuong_tin,
    locations="ten",
    color="matdo",
    range_color=(0, dan_so["matdo"].max()),
    featureidkey="properties.name",
    map_style="carto-positron",
    color_continuous_scale="viridis",
    zoom=11,
    opacity=0.5,
    center = {"lon": 105.87326311496878, "lat": 20.847436458212155},
    labels = {"name": "tên", "density": "mật độ"},
)
fig.update_layout(coloraxis_colorbar_title_text="Mật độ dân số (người/km2)")
fig.update_layout(margin={"r":0,"t":0,"l":0,"b":0})
fig.show()
{% endhighlight %}

Và giờ ta được thành quả.

![Bản đồ dân số Thường Tín](/assets/images/thuong-tin.png)

Tất cả code trong bài viêt.

{% highlight python %}
import json
import pandas as pd
import plotly.express as px

# Đọc file ban-do-thuong-tin.json
with open("ban-do-thuong-tin.json") as f:
    ban_do_thuong_tin = json.load(f)

# Đọc file dan-so-thuong-tin.csv
dan_so = pd.read_csv("dan-so-thuong-tin.csv")
# Vì trong dữ liệu chưa có cột mật độ nên giờ ta cần tính
dan_so["matdo"] = dan_so["danso"] / (dan_so["dientich"]*0.01)

fig = px.choropleth_map(
    dan_so,
    geojson=ban_do_thuong_tin,
    locations="ten",
    color="matdo",
    range_color=(0, dan_so["matdo"].max()),
    featureidkey="properties.name",
    map_style="carto-positron",
    color_continuous_scale="viridis",
    zoom=11,
    opacity=0.5,
    center = {"lon": 105.87326311496878, "lat": 20.847436458212155},
    labels = {"name": "tên", "density": "mật độ"},
)
fig.update_layout(coloraxis_colorbar_title_text="Mật độ dân số (người/km2)")
fig.update_layout(margin={"r":0,"t":0,"l":0,"b":0})
fig.show()
{% endhighlight %}
