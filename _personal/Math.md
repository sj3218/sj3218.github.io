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
{% include elements/button.html link="https://drive.google.com/drive/folders/1EcWeskiSsk8NeywwD8uC2kODcikfnT1a" text="Source Code 링크 이동" block=true %}

<br>

### **Curve**

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

### **K means Clustering 알고리즘**

###### Data Science 수학 수업에서 K-means Clustering 알고리즘에 대해서 배우고 이 알고리즘을 이용하여 입력 받은 사진이 k값에 따라 어떤 결과 값을 갖는지 테스트 해보고자 한 프로젝트 입니다. 
###### 이 프로젝트는 C++과 OpenCV를 사용하여 구현였습니다. 인풋된 이미지를 받아와 RGB를 저장하고 k 값이 3부터 차례대로 늘어나며 어떤 결과 값을 보여주는지 이미지를 아웃풋 시킵니다. k 값은 이미지에서 보여줄 수 있는 색깔의 종류 갯수입니다. 사진에 보이는 것 처럼 k가 3일 때는 세개의 색상만을 보여줍니다.

<!--
{% capture carousel_images %}
/assets/images/math/k_means/mario_3.jpg
/assets/images/math/k_means/mario_4.jpg
/assets/images/math/k_means/mario_5.jpg
/assets/images/math/k_means/mario_6.jpg
/assets/images/math/k_means/mario_7.jpg
/assets/images/math/k_means/mario_8.jpg
/assets/images/math/k_means/mario_9.jpg
/assets/images/math/k_means/mario_10.jpg
{% endcapture %}
{% include elements/carousel.html %}

-->

<style>
.carousel-inner img {
    max-height: 50%;  /* 원하는 높이로 조정 */
    width: auto;      /* 비율에 따라 자동 조정 */
    height: auto;     /* 비율에 따라 자동 조정 */
    object-fit: cover; /* 이미지 비율을 유지하며 잘라내기 */
}
</style>

<div id="carousel2" class="carousel slide" data-ride="carousel">
    <div class="carousel-inner">
        <div class="carousel-item active">
            <img src="/assets/images/math/k_means/mario_3.jpg" class="d-block w-100" alt="">
        </div>
        <div class="carousel-item">
            <img src="/assets/images/math/k_means/mario_4.jpg" class="d-block w-100" alt="">
        </div>
        <div class="carousel-item">
            <img src="/assets/images/math/k_means/mario_5.jpg" class="d-block w-100" alt="">
        </div>
        <div class="carousel-item">
            <img src="/assets/images/math/k_means/mario_6.jpg" class="d-block w-100" alt="">
        </div>
        <div class="carousel-item">
            <img src="/assets/images/math/k_means/mario_7.jpg" class="d-block w-100" alt="">
        </div>
        <div class="carousel-item">
            <img src="/assets/images/math/k_means/mario_8.jpg" class="d-block w-100" alt="">
        </div>
        <div class="carousel-item">
            <img src="/assets/images/math/k_means/mario_9.jpg" class="d-block w-100" alt="">
        </div>
        <div class="carousel-item">
            <img src="/assets/images/math/k_means/mario_10.jpg" class="d-block w-100" alt="">
        </div>
    </div>
    <a class="carousel-control-prev" href="#carousel2" role="button" data-slide="prev">
        <span class="carousel-control-prev-icon" aria-hidden="true"></span>
        <span class="sr-only">Previous</span>
    </a>
    <a class="carousel-control-next" href="#carousel2" role="button" data-slide="next">
        <span class="carousel-control-next-icon" aria-hidden="true"></span>
        <span class="sr-only">Next</span>
    </a>
</div>


<br>

###### 마리오 원본 사진
<img src="/assets/images/math/k_means/original_mario.jpg" alt="alt text" style="width: auto; height: 50%;">


<br>

## **배운 점**
###### Curve 프로젝트를 통해 Newton Formula에 관해서 더 연구할 수 있었고 직접 적용하여 구현할 수 있었습니다. 이 프로젝트를 진행하며 높은 차수의 다항식을 사용할 수록 계산 오차가 크게 증가하거나 속도 저하에 대한 어려움이 있었습니다. 예를 들어 데이터 점의 개수가 많을 수록 계산 과정에서 소수점 오류가 발생하거나 속도가 느려졌었습니다. 이를 해결 하기 위해서 이미 계산 했던 값들은 dp[]라는 배열에 저장을 하였고 이 데이터를 처리하는 타입들을 int에서 float값으로 바꾸는 등을 시도하였습니다. 결과적으로 프로젝트의 수치적으로 안정성을 높일 수 있었습니다.

<br>

###### K-means Clustering 프로젝트를 통해 k값을 어떻게 설정하느냐에 따라 이미지의 표현 방식이 크게 달라진다는 것을 경험했습니다. K 값이 작을 수록 적은 색상으로 이미지를 표현하게 되어 단순화된 형태로 나타났고 K값을 늘리면 더 세밀한 색상 표현이 가능해져 디테일이 살아났습니다.
###### 이 프로젝트에서는 마리오 사진을 선택하여 K값에 따라 결과값이 좋게 보여졌지만, 색깔을 많이 가지고 있거나 복잡한 패턴을 가진 이미지에서는 K 값에 따라 적합하지 않은 색상 구분이 발생하였습니다. 이를 통해 항상 K-means가 최적의 군집화를 제공하지 않는 다는 점도 알 수 있었습니다. 이미지의 특성에 따라 다른 기법을 고려해 보아야 한다는 점을 배운 것 같습니다.
<br>
<br>
<br>
<br>
<br>
<br>