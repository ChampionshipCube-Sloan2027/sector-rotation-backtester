# Contributing to Sector Rotation Backtester

Thank you for your interest in contributing! This project is open to improvements, bug fixes, and new features.

## How to Contribute

### Reporting Bugs

1. Check if the bug is already reported in [Issues](https://github.com/yourusername/sector-rotation-backtester/issues)
2. If not, create a new issue with:
   - **Title**: Brief description of the bug
   - **Description**: Detailed steps to reproduce
   - **Expected behavior**: What should happen
   - **Actual behavior**: What actually happens
   - **Environment**: Browser, OS, date range tested

Example:
```
Title: Sharpe ratio calculation incorrect for short periods

Description: When backtesting with less than 30 trading days, 
Sharpe ratio shows NaN instead of a valid number.

Steps to reproduce:
1. Set start date to 2024-12-01
2. Set end date to 2024-12-10
3. Click "Run backtest"
4. Observe NaN in Sharpe ratio metric

Expected: Valid Sharpe ratio or "Insufficient data" message
Actual: Shows NaN
```

### Suggesting Features

1. Check [Issues](https://github.com/yourusername/sector-rotation-backtester/issues) for existing requests
2. Create a new issue with:
   - **Title**: Feature name
   - **Use case**: Why this is valuable
   - **Proposed implementation**: How you'd implement it (optional)

Example:
```
Title: Add Calmar Ratio metric

Use case: Calmar ratio (return / max drawdown) is a standard 
metric in quant finance. Would help analysts compare strategies 
more holistically.

Proposed implementation: Add calculation in metrics.js, 
display in metrics-grid with other ratios.
```

### Code Contributions

#### 1. Fork the Repository
```bash
# On GitHub, click "Fork" button
# Then clone your fork:
git clone https://github.com/yourusername/sector-rotation-backtester.git
cd sector-rotation-backtester
```

#### 2. Create a Feature Branch
```bash
git checkout -b feature/your-feature-name
# or for bugs:
git checkout -b fix/bug-description
```

Branch naming convention:
- `feature/add-calmar-ratio` — new feature
- `fix/sharpe-nan-bug` — bug fix
- `docs/update-readme` — documentation
- `refactor/split-metrics-module` — code cleanup

#### 3. Make Your Changes

Keep changes focused:
- One feature or bug per PR
- Follow existing code style
- Add comments for non-obvious logic
- Test in multiple browsers

**Code style**:
```javascript
// ✓ Good
function calculateSharpeRatio(returns, riskFreeRate = 0.045) {
  const meanReturn = returns.reduce((a, b) => a + b, 0) / returns.length;
  const variance = returns.reduce((sum, r) => sum + (r - meanReturn) ** 2, 0) / returns.length;
  return (meanReturn - riskFreeRate) / Math.sqrt(variance);
}

// ✗ Avoid
function calcSharpe(r, rf) {
  let m = r.reduce((a,b)=>a+b)/r.length;
  return (m-rf)/Math.sqrt(r.reduce((s,x)=>(x-m)**2)/r.length);
}
```

#### 4. Test Your Changes

- Open `index.html` in your browser
- Run various backtests with different inputs
- Test edge cases:
  - Very short date ranges (< 10 days)
  - Very long date ranges (> 20 years)
  - Different rotation rules
  - Extreme VIX values (5 and 80)
  - Extreme unemployment values (3.5 and 6.5)

#### 5. Commit with Clear Messages

```bash
git add .
git commit -m "feature: add Calmar ratio metric

- Calculate return/max drawdown ratio
- Display in metrics dashboard
- Add tooltip explaining formula
- Handles edge case when maxDD = 0"
```

Format: `<type>: <description>`
- `feature`: New functionality
- `fix`: Bug fix
- `docs`: Documentation changes
- `refactor`: Code restructuring
- `test`: Adding tests
- `style`: Code style (formatting, missing semicolons)

#### 6. Push to Your Fork
```bash
git push origin feature/your-feature-name
```

#### 7. Create a Pull Request

1. Go to [Pull Requests](https://github.com/yourusername/sector-rotation-backtester/pulls)
2. Click "New Pull Request"
3. Select `yourusername/sector-rotation-backtester` as source
4. Fill in the template:

```markdown
## Description
Brief explanation of what this PR does

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Documentation update
- [ ] Refactoring

## Testing
- [ ] Tested with short date range (< 30 days)
- [ ] Tested with long date range (> 5 years)
- [ ] Tested all rotation rule types
- [ ] Tested in Chrome
- [ ] Tested in Firefox
- [ ] Tested in Safari

## Checklist
- [ ] My code follows the project's style guide
- [ ] I've added comments for non-obvious logic
- [ ] I've tested the changes thoroughly
- [ ] I haven't introduced new bugs (to my knowledge)
```

## Architecture & Key Files

### Core Files

- **index.html** — Main application file, contains React app and Chart.js setup
- **src/backtester.js** — `calculateReturns()` function (the engine)
- **src/metrics.js** — `calculateMetrics()` function (statistics)
- **src/dataGenerator.js** — `generateHistoricalData()` function (simulated data)

### Flow

```
User Input → rotationRule object
     ↓
generateHistoricalData() → array of { date, spy, vix, unemployment, sectors }
     ↓
calculateReturns() → { dates, strategyValue, spyValue, drawdown, allocation }
     ↓
calculateMetrics() → { strategyReturn, sharpe, maxDD, winRate, ... }
     ↓
renderCharts() → Chart.js visualizations
```

### Adding a New Rotation Rule

1. Open `index.html`
2. In the form input section, add an option:
   ```jsx
   <option value="my_rule">My Custom Rule</option>
   ```

3. In `calculateReturns()`, add the logic:
   ```javascript
   if (rotationRule.type === 'my_rule') {
     if (/* your condition */) {
       allocation = { 'SPY': 0.1, 'Defensive': 0.6, 'Growth': 0.2, 'Other': 0.1 };
     } else {
       allocation = { 'SPY': 0.1, 'Defensive': 0.2, 'Growth': 0.6, 'Other': 0.1 };
     }
   }
   ```

4. Test with various inputs
5. Submit a PR with explanation of the rule's logic

## Project Roadmap

High-priority contributions:
1. **Real data integration** — Replace simulated data with yfinance
2. **Transaction cost modeling** — Add slippage and commissions
3. **Additional metrics** — Calmar ratio, Sortino ratio, information ratio
4. **Walk-forward validation** — Out-of-sample testing
5. **Export functionality** — Save results as CSV/PDF

## Questions?

- Open a [Discussion](https://github.com/yourusername/sector-rotation-backtester/discussions)
- Email: [your email]
- Check [existing issues](https://github.com/yourusername/sector-rotation-backtester/issues)

## Code of Conduct

This project adheres to the [Contributor Covenant](https://www.contributor-covenant.org/).

By participating, you agree to:
- Be respectful to other contributors
- Provide constructive feedback
- Focus on the code, not the person
- Help create an inclusive environment

---

**Thank you for contributing!** 🎉
