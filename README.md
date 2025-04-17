# prjct
import pandas as pd
from dash import Dash, dcc, html, Input, Output
import plotly.express as px

# df = pd.read_csv('your_data.csv')

# ['patient_id', 'age', 'gender', 'condition', 'heart_rate', 'blood_pressure', 'last_update']

df = pd.read_csv("C:\Users\makss\OneDrive\Bureaublad\patients.csv")

app = Dash(__name__)

app.layout = html.Div([
    html.H1("Расширенная панель мониторинга пациентов", style={"textAlign": "center", "color": "#2c3e50"}),

    html.Div([
        dcc.Dropdown(
            id="gender-filter",
            options=[{"label": g, "value": g} for g in ["All", "Male", "Female"]],
            value="All",
            placeholder="Фильтр по полу",
            style={"marginBottom": "10px"}
        ),
        dcc.Dropdown(
            id="condition-filter",
            options=[{"label": c, "value": c} for c in ["All", "Stable", "Critical", "Recovering"]],
            value="All",
            placeholder="Фильтр по состоянию",
            style={"marginBottom": "20px"}
        ),
        dcc.RangeSlider(
            id="age-slider",
            min=18,
            max=90,
            step=5,
            marks={i: str(i) for i in range(20, 91, 10)},
            value=[18, 90]
        )
    ], style={"width": "30%", "margin": "20px", "padding": "20px", "boxShadow": "0px 0px 5px #ccc"}),

    html.Div([
        dcc.Graph(id="vital-signs-plot", style={"height": "400px"}),
        html.Div([
            dcc.Graph(id="patient-distribution", style={"width": "45%", "display": "inline-block"}),
            dcc.Graph(id="hr-distribution", style={"width": "45%", "display": "inline-block"})
        ]),
        html.Div(id="advanced-stats", style={
            "padding": "20px",
            "marginTop": "20px",
            "border": "1px solid #3498db",
            "borderRadius": "5px",
            "backgroundColor": "#f9f9f9"
        })
    ])
])

@app.callback(
    Output("vital-signs-plot", "figure"),
    Output("patient-distribution", "figure"),
    Output("hr-distribution", "figure"),
    Output("advanced-stats", "children"),
    Input("gender-filter", "value"),
    Input("condition-filter", "value"),
    Input("age-slider", "value")
)
def update_dashboard(selected_gender, selected_condition, age_range):
    if df.empty:
        return {}, {}, {}, "Загрузите данные пациентов для отображения аналитики"
    
    filtered_df = df[
        (df["age"] >= age_range[0]) &
        (df["age"] <= age_range[1])
    ]

    if selected_gender != "All":
        filtered_df = filtered_df[filtered_df["gender"] == selected_gender]
    if selected_condition != "All":
        filtered_df = filtered_df[filtered_df["condition"] == selected_condition]

    fig1 = px.scatter(
        filtered_df,
        x="last_update",
        y="heart_rate",
        color="condition",
        size="age",
        hover_data=["blood_pressure"],
        title="Динамика сердечного ритма",
        template="plotly_white"
    ) if not filtered_df.empty else {}

    fig2 = px.pie(
        filtered_df,
        names="condition",
        hole=0.4,
        title="Распределение состояний"
    ) if not filtered_df.empty else {}

    fig3 = px.histogram(
        filtered_df,
        x="heart_rate",
        nbins=20,
        color="gender",
        title="Распределение пульса"
    ) if not filtered_df.empty else {}

    stats = [
        html.H4("Статистика:", style={"color": "#3498db"}),
        html.P(f"Пациентов: {len(filtered_df)}"),
        html.P(f"Средний возраст: {filtered_df['age'].mean():.1f} лет"),
        html.P(f"Средний пульс: {filtered_df['heart_rate'].mean():.1f} уд/мин")
    ] if not filtered_df.empty else "Нет данных для отображения"

    return fig1, fig2, fig3, stats

if __name__ == "__main__":
    app.run(debug=True)
