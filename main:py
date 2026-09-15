import streamlit as st
import pandas as pd
import numpy as np
import plotly.graph_objects as go

# ────────────────────────────────
# 기본 설정
# ────────────────────────────────
st.set_page_config(
    page_title="서울 100년 기온 변화",
    page_icon="🌡️",
    layout="centered",
)

DATA_URL = "https://raw.githubusercontent.com/greatsong/modudata/main/data/seoul.csv"


@st.cache_data(show_spinner="데이터를 불러오는 중이에요...")
def load_data(url: str) -> pd.DataFrame:
    df = pd.read_csv(url)
    df["날짜"] = pd.to_datetime(df["날짜"])
    df["연도"] = df["날짜"].dt.year
    return df


@st.cache_data(show_spinner=False)
def make_yearly(df: pd.DataFrame) -> pd.DataFrame:
    yearly = (
        df.groupby("연도")
        .agg(
            연평균기온=("평균기온", "mean"),
            연평균최저기온=("최저기온", "mean"),
            연평균최고기온=("최고기온", "mean"),
            관측일수=("평균기온", "count"),
        )
        .reset_index()
    )
    # 자료가 부족한 해(관측일수가 너무 적은 해)는 제외해서 그래프가 왜곡되지 않게 함
    yearly = yearly[yearly["관측일수"] >= 300].drop(columns="관측일수")
    return yearly


# ────────────────────────────────
# 제목 / 소개
# ────────────────────────────────
st.title("🌡️ 서울, 100년 동안 얼마나 더워졌을까?")
st.markdown(
    """
서울의 **일별 기온 관측 데이터**를 바탕으로, 연평균 기온이
지난 100여 년 동안 어떻게 변해왔는지 한눈에 보여주는 페이지예요.
"""
)

# ────────────────────────────────
# 데이터 로드
# ────────────────────────────────
with st.spinner("서울 기온 데이터를 불러오는 중..."):
    raw_df = load_data(DATA_URL)
    yearly_df = make_yearly(raw_df)

start_year = int(yearly_df["연도"].min())
end_year = int(yearly_df["연도"].max())
first_temp = yearly_df.iloc[0]["연평균기온"]
last_temp = yearly_df.iloc[-1]["연평균기온"]
temp_diff = last_temp - first_temp

# ────────────────────────────────
# 핵심 요약 지표
# ────────────────────────────────
st.markdown("### 📌 핵심 요약")
col1, col2, col3 = st.columns(3)
col1.metric(f"{start_year}년 연평균기온", f"{first_temp:.1f} °C")
col2.metric(f"{end_year}년 연평균기온", f"{last_temp:.1f} °C")
col3.metric("전체 변화", f"{temp_diff:+.1f} °C")

st.markdown(
    f"""
> **{start_year}년**부터 **{end_year}년**까지 약 **{end_year - start_year}년** 동안,
> 서울의 연평균 기온은 **{temp_diff:+.1f}°C** 변했어요.
"""
)

# ────────────────────────────────
# 그래프: 연평균 기온 변화 + 추세선
# ────────────────────────────────
st.markdown("### 📈 연평균 기온 변화 그래프")

show_range = st.checkbox("최저·최고기온도 함께 보기", value=False)

years = yearly_df["연도"].values
avg_temp = yearly_df["연평균기온"].values

# 추세선 계산 (1차 선형 회귀)
coeffs = np.polyfit(years, avg_temp, 1)
trend = np.polyval(coeffs, years)
change_per_10yr = coeffs[0] * 10

fig = go.Figure()

if show_range:
    fig.add_trace(
        go.Scatter(
            x=yearly_df["연도"],
            y=yearly_df["연평균최고기온"],
            mode="lines",
            name="연평균 최고기온",
            line=dict(color="#FF9E9E", width=1.5, dash="dot"),
        )
    )
    fig.add_trace(
        go.Scatter(
            x=yearly_df["연도"],
            y=yearly_df["연평균최저기온"],
            mode="lines",
            name="연평균 최저기온",
            line=dict(color="#9ECBFF", width=1.5, dash="dot"),
        )
    )

fig.add_trace(
    go.Scatter(
        x=yearly_df["연도"],
        y=avg_temp,
        mode="lines+markers",
        name="연평균기온",
        line=dict(color="#F4A300", width=2.5),
        marker=dict(size=4),
    )
)

fig.add_trace(
    go.Scatter(
        x=years,
        y=trend,
        mode="lines",
        name="추세선 (전반적 경향)",
        line=dict(color="#D64545", width=3, dash="dash"),
    )
)

fig.update_layout(
    xaxis_title="연도",
    yaxis_title="기온 (°C)",
    hovermode="x unified",
    height=480,
    legend=dict(orientation="h", yanchor="bottom", y=1.02, xanchor="right", x=1),
    margin=dict(l=10, r=10, t=40, b=10),
    font=dict(size=15),
)

st.plotly_chart(fig, use_container_width=True)

st.info(
    f"📐 추세선 기준으로, 서울의 연평균 기온은 **10년마다 약 {change_per_10yr:+.2f}°C** "
    "씩 변해온 셈이에요. (직선으로 단순화한 경향이라 실제 등락은 더 복잡해요!)"
)

# ────────────────────────────────
# 가장 더웠던 해 / 추웠던 해
# ────────────────────────────────
st.markdown("### 🏆 가장 더웠던 해 & 가장 추웠던 해")

hottest = yearly_df.loc[yearly_df["연평균기온"].idxmax()]
coldest = yearly_df.loc[yearly_df["연평균기온"].idxmin()]

hcol, ccol = st.columns(2)
with hcol:
    st.success(
        f"🔥 **가장 더웠던 해**\n\n"
        f"**{int(hottest['연도'])}년** — 연평균 {hottest['연평균기온']:.1f}°C"
    )
with ccol:
    st.info(
        f"❄️ **가장 추웠던 해**\n\n"
        f"**{int(coldest['연도'])}년** — 연평균 {coldest['연평균기온']:.1f}°C"
    )

# ────────────────────────────────
# 원본 데이터 살펴보기
# ────────────────────────────────
with st.expander("🔍 연도별 데이터 표로 보기"):
    st.dataframe(
        yearly_df.round(1).sort_values("연도", ascending=False),
        use_container_width=True,
        hide_index=True,
    )

st.caption(
    "데이터 출처: 기상청 서울(지점번호 108) 일별 기온 관측 자료 "
    "(github.com/greatsong/modudata)"
)
