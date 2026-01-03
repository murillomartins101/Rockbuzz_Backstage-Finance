# Copilot Instructions for Rockbuzz Backstage Finance

## Project Overview

Rockbuzz Backstage Finance is a professional financial dashboard application built with Streamlit for managing and visualizing financial data. The application provides comprehensive financial analytics, transaction management, and reporting capabilities with integration to Google Sheets for data persistence.

## Tech Stack

- **Python 3.11+**: Primary programming language
- **Streamlit 1.38.0+**: Web application framework for the dashboard UI
- **Pandas 2.2.2+**: Data manipulation and analysis
- **Plotly 5.22.0+**: Interactive data visualizations
- **NumPy 1.26.4+**: Numerical computing
- **gspread 6.1.2+**: Google Sheets API integration
- **oauth2client 4.1.3+**: OAuth authentication for Google Sheets

## Project Structure

- `rockbuzz_backstage_finance.py`: Main application file containing all dashboard logic, data processing, and UI components (~2000+ lines)
- `requirements.txt`: Python dependencies
- `.devcontainer/`: Development container configuration for consistent development environment
- `EFFICIENCY_REPORT.md`: Documentation of known performance issues and potential optimizations

## Key Features & Modules

1. **Financial Data Management**: Transaction tracking, categorization, and management
2. **KPI Dashboard**: Key performance indicators with visual cards (revenue, expenses, balance, etc.)
3. **Visualization**: Interactive charts using Plotly (line charts, bar charts, waterfall charts)
4. **Period Analysis**: Monthly, quarterly, yearly financial analysis with custom date ranges
5. **Google Sheets Integration**: Optional data persistence and synchronization
6. **Rateio & Cost Centers**: Cost allocation and center management functionality
7. **Data Import/Export**: CSV and Excel file upload/download capabilities

## Coding Conventions & Style

### Python Style
- Follow PEP 8 style guide for Python code
- Use type hints for function parameters and return values (see `from __future__ import annotations`)
- Use descriptive variable names in Portuguese (e.g., `categoria`, `valor`, `descricao`) as this is a Brazilian application
- Prefer f-strings for string formatting
- Keep functions focused and single-purpose

### Streamlit Patterns
- Use `st.set_page_config()` at the top of the file (already configured)
- Employ Streamlit's column layout for responsive design (`st.columns()`)
- Use session state (`st.session_state`) for maintaining state across reruns
- Apply custom CSS through `st.markdown()` for consistent styling
- Use `@st.cache_data` or `@st.cache_resource` for expensive computations

### Data Processing
- Use pandas vectorized operations instead of `iterrows()` for better performance
- Handle missing values explicitly with `pd.notna()` or `.fillna()`
- Normalize column names to lowercase and remove special characters
- Use the `brl()` function for formatting Brazilian Real currency values
- Apply the `normalize_valor_series()` function for cleaning monetary values from strings

### Error Handling
- Use try-except blocks for optional dependencies (see Google Sheets integration pattern)
- Provide graceful degradation when optional features are unavailable
- Display user-friendly error messages through Streamlit's UI components

## Running the Application

### Local Development
```bash
# Install dependencies
pip install -r requirements.txt

# Run the application
streamlit run rockbuzz_backstage_finance.py
```

### Using DevContainer
The project includes a DevContainer configuration that automatically:
- Sets up Python 3.11 environment
- Installs all dependencies from requirements.txt
- Starts the Streamlit server on port 8501
- Configures CORS and XSRF protection for development

## Testing & Validation

- **Manual Testing**: Run the Streamlit app and test UI interactions
- **Data Validation**: Verify calculations match expected financial formulas
- **Visual Inspection**: Check that charts render correctly and KPIs display accurate values
- **Edge Cases**: Test with empty data, missing values, and extreme date ranges

## Important Considerations

### Performance
- Be aware of DataFrame operations on large datasets (thousands of transactions)
- Avoid `iterrows()` in favor of vectorized operations (see EFFICIENCY_REPORT.md)
- Consider caching expensive data transformations with `@st.cache_data`

### Security
- Never commit Google Sheets credentials or API keys
- Use environment variables or Streamlit secrets for sensitive configuration
- Validate and sanitize user inputs from file uploads

### Known Issues
- Critical bug at line 688: Incorrect DataFrame filtering syntax (documented in EFFICIENCY_REPORT.md)
- Performance bottleneck with `iterrows()` at lines 729-733 (documented in EFFICIENCY_REPORT.md)

## Common Patterns

### Currency Formatting
```python
brl(1000)  # Returns: "R$ 1.000,00"
```

### Date Formatting
```python
fmt_brdate(date_series)  # Converts to Brazilian date format (dd/mm/yyyy)
```

### KPI Card Rendering
```python
kpis = [
    {"label": "Receitas", "value": brl(10000), "color": "green"},
    {"label": "Despesas", "value": brl(5000), "color": "red"}
]
st.markdown(render_kpi_cards(kpis), unsafe_allow_html=True)
```

### Google Sheets Integration
```python
if GS_AVAILABLE:
    # Use Google Sheets functionality
    df = read_sheet("lancamentos")
else:
    # Fallback to local data
    pass
```

## Best Practices for Modifications

1. **Maintain UI Consistency**: Follow the established dark theme with Inter font family
2. **Preserve Data Integrity**: Always validate data transformations don't lose information
3. **Keep Backwards Compatibility**: Ensure changes don't break existing Google Sheets integration
4. **Document Complex Logic**: Add comments for non-obvious financial calculations
5. **Test with Real Data**: Use realistic financial data patterns for testing
6. **Responsive Design**: Ensure UI elements work on different screen sizes
7. **Performance First**: Optimize data operations for large transaction volumes

## UI/UX Guidelines

- Use dark cards (#1a1a2e background) for KPI displays
- Color coding: Green for revenue/positive, Red for expenses/negative, Blue for neutral, Yellow/Gold for highlights
- Maintain consistent spacing and padding (1rem, 1.5rem standard)
- Use Inter font with appropriate weights (300-800)
- Provide clear labels and tooltips for user guidance
- Show loading states for long-running operations

## Language & Localization

- UI labels and messages should be in Portuguese (Brazilian)
- Date format: dd/mm/yyyy (Brazilian standard)
- Currency format: R$ with thousands separator (.) and decimal separator (,)
- Number formatting follows Brazilian conventions

## When Making Changes

- Always review EFFICIENCY_REPORT.md for known issues before making changes
- Test changes with different data scenarios (empty, small, large datasets)
- Verify Google Sheets sync if modifying data structures
- Check responsive behavior on different screen sizes
- Ensure custom CSS doesn't break Streamlit's default styling
- Validate that financial calculations remain accurate

## Questions or Clarifications

When unsure about:
- Financial calculation logic → Review existing functions like `calcular_ticket_medio()`
- Data structure → Check the DataFrame column names and types in the main data loading section
- UI styling → Refer to the custom CSS section at the top of the file
- Google Sheets schema → Review the `ensure_ws_with_header()` function
