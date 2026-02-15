# Contributing to EduQuest

Thank you for your interest in contributing to EduQuest! This document provides guidelines for contributing to the project.

## Getting Started

1. Fork the repository
2. Clone your fork: `git clone https://github.com/yourusername/eduquest.git`
3. Create a feature branch: `git checkout -b feature/your-feature-name`
4. Make your changes
5. Run tests: `pytest` (backend) and `npm test` (frontend)
6. Commit your changes: `git commit -m "Add your feature"`
7. Push to your fork: `git push origin feature/your-feature-name`
8. Open a Pull Request

## Development Setup

### Backend
```bash
cd backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
cp ../.env.example .env
# Edit .env with your AWS credentials
python main.py
```

### Frontend
```bash
cd frontend
npm install
npm run dev
```

## Code Style

- **Python**: Follow PEP 8, use type hints
- **TypeScript**: Follow ESLint configuration
- **Commits**: Use conventional commits format

## Testing

- Write unit tests for new features
- Ensure all tests pass before submitting PR
- Add property-based tests for critical logic

## Pull Request Process

1. Update documentation if needed
2. Add tests for new features
3. Ensure CI/CD passes
4. Request review from maintainers
5. Address review feedback

## Questions?

Open an issue or reach out to the maintainers.
