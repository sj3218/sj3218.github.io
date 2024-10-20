---
layout: page
name: 수학 프로그래밍
tools: [C++, C#, OpenCV, K-means]
image: "/assets/images/background/math.JPG"
---

# Math Programming

<br>
{% include elements/video.html id="LgG7_HTG_Jk" %}

<br>

### Curve

###### Interpolating Polynomial 함수를 3D 버전으로 구현하였습니다. 이 프로젝트를 진행하기 위해 [**Newton Formula**](https://en.wikipedia.org/wiki/Newton_polynomial)에 대해서 공부를 하였습니다.
###### 먼저 InitializeCoefficientForNewton() 함수를 만들어 각 x,y,z 축마다 Newton Formula를 위한 계수를 구하였습니다. 그리고 이후에 점이 추가 될 때마다 이 함수를 불러 계수를 업데이트 하도록 하였습니다.
```c++
else if (type == 2) //z축
{
    for (index = 0; index < size; ++index)
    {
        if (divided_difference_z[index] == 0)
        {
            divided_difference_z[index] = (double)points[index].z;
        }
    }

    newtonForm_z[newton_index++] = divided_difference_z[0];

    for (int i = 1; i < size; ++i)
    {
        index = 20 * i;
        for (int j = 0; j < size - i; ++j)
        {
            if (divided_difference_z[index] == 0)
            {
                index_left = 20 * (i - 1) + j;
                divided_difference_z[index] = (divided_difference_z[index_left + 1] - divided_difference_z[index_left]) / i;
            }
            if (j == 0)
            {
                newtonForm_z[newton_index++] = divided_difference_z[index];
            }
            ++index;
        }
    }
}
```
###### 곡선을 그리기 위해 CalculationSplinePoint() 함수를 작성하였습니다. 각 점 마다 Newton 방정식을 사용하여 곡선의 위치를 저장하였습니다.
```c++
void SplinePath::CalculationSplinePoint()
{
    spline_pts.clear();
    int size = points.size();
    float step = points.size() * 5;
    float alpha = 1 / step;

    glm::vec3 pos;
    glm::vec3 previous_pos;

    previous_pos.x = NewtonForm(0, 0);
    previous_pos.y = NewtonForm(1, 0);
    previous_pos.z = NewtonForm(2, 0);

    if (size != 1)
    {
        for (double t = alpha; t < size -1; t += alpha)
        {
            spline_pts.push_back(previous_pos);

            pos.x = NewtonForm(0, t);
            pos.y = NewtonForm(1, t);
            pos.z = NewtonForm(2, t);

            previous_pos = pos;

            spline_pts.push_back(pos);
        }

        spline_pts.push_back(previous_pos);
        pos.x = NewtonForm(0, size - 1);
        pos.y = NewtonForm(1, size - 1);
        pos.z = NewtonForm(2, size - 1);
        spline_pts.push_back(pos);
    }
}
```
###### NewtonForm()함수에서는 실제 Newton 방정식을 수행하는 함수입니다. 각 축마다 계산을 하여 나온 값을 리턴시켰습니다.
```c++
if (type == 0)//x축
{
    if (size == 1)
    {
        return (double)points[0].x;
    }

    p_value = newtonForm_x[0];

    for (int i = 1; i < size; ++i)
    {
        t_value *= (t - i + 1);
        p_value += newtonForm_x[i] * (double)t_value;
    }
}
```


{% capture carousel_images %}
/assets/images/math/Bezier_curve.JPG
/assets/images/math/deboor.JPG
/assets/images/math/Interpolating_polynomials.JPG
/assets/images/math/Polynomial_Functions.JPG
/assets/images/math/3d_spline.JPG
{% endcapture %}
{% include elements/carousel.html %}

<br>

### K means Clustering 알고리즘

###### C++과 OpenCV를 사용하여 구현였습니다. 인풋된 이미지를 받아와 RGB를 저장하고 k 값이 3부터 차례대로 늘어나며 어떤 결과 값을 보여주는지 이미지를 아웃풋 시킵니다. k 값은 이미지에서 보여줄 수 있는 색깔의 종류 갯수입니다. 사진에 보이는 것 처럼 k가 3일 때는 세개의 색상만을 보여줍니다.

{% capture carousel_images %}
/assets/images/math/k_means/mario_3.JPG
/assets/images/math/k_means/mario_4.JPG
/assets/images/math/k_means/mario_5.JPG
/assets/images/math/k_means/mario_6.JPG
/assets/images/math/k_means/mario_7.JPG
/assets/images/math/k_means/mario_8.JPG
/assets/images/math/k_means/mario_9.JPG
/assets/images/math/k_means/mario_10.JPG
{% endcapture %}
{% include elements/carousel.html %}
<br>

###### 마리오 원본 사진
![alt text](
/assets/images/math/k_means/original_mario.JPG)
