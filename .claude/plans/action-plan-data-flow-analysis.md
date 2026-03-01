# Action Plan Data Flow Analysis — CitedBy.ai

**Date**: 2026-02-15  
**Project**: CitedBy.ai (SaaS X-Final-Rick-Test)  
**Goal**: Understand how recommendations → generated solutions flow works, and how to make Directory join conditional on ALL solutions being validated.

---

## 1. COMPLETE DATA FLOW: History → Projects → ActionPlan → Recommendations → GeneratedSolutions

### 1.1 Root State (App.tsx)

```
App.tsx
├── history: AnalysisResult[] (in state, line 62)
│   └── Loaded from DB on mount (line 66)
│   └── Updated by handleAnalyze() when new analysis created (line 231)
│
├── analysisData: AnalysisResult | null (line 61)
│   └── Set by handleAnalyze() or handleViewAnalysis()
│
└── handleSaveSolution() callback (line 283)
    └── Updates analysisData.generatedSolutions
    └── Updates history[index].generatedSolutions
    └── Persists to DB via db.update()
```

**Key State**: 
- `history` is the master list maintained in App.tsx
- Each `AnalysisResult` contains:
  - `url`, `timestamp`, `globalScore`, `categories`, `recommendations`
  - `aiCitation` data
  - **`generatedSolutions`**: `Record<string, GeneratedSolution & { timestamp: string }>` (optional)

---

### 1.2 Navigation to Projects (Monitoring AEO)

**Projects.tsx** receives:
```typescript
interface ProjectsProps {
  history: AnalysisResult[];                    // From App (line 13)
  onSaveSolution: (analysisId, recId, solution) => void;  // From App (line 19)
  // ... other props
}
```

**How Projects finds the current analysis**:
```
Projects.tsx (line 41)
├── Receives history[] from App
│
├── const projects = useMemo() (line 58)
│   └── Deduplicates history by hostname (latest first)
│   └── Converts to Project[] format for display
│
├── const projectHistory = useMemo() (line 109)
│   └── Filters history for selectedProject.name
│   └── Sorted by timestamp (newest first)
│
└── const activeAnalysis (line 124)
    └── selectedAnalysisState || projectHistory[0]
    └── The currently displayed AnalysisResult
```

---

### 1.3 Navigation to ActionPlan Tab

When user clicks "Plan d'Action" tab (Projects.tsx line 424):

```typescript
handleSwitchToActionPlan() {
  if (activeAnalysis) {
    setActiveTab('action-plan');  // line 184
  }
}
```

This renders ActionPlan component (Projects.tsx line 244):
```tsx
<ActionPlan 
  data={activeAnalysis}                      // ← The selected AnalysisResult
  user={user}
  onDeductCredits={onDeductCredits}
  onSaveSolution={onSaveSolution}           // ← From App.tsx
  onNavigateToAnalysis={onNewAnalysis}
  onNavigateToRecharge={onNavigateToRecharge}
/>
```

---

### 1.4 ActionPlan Component Logic

**Input**: `data: AnalysisResult`

**Structure** (ActionPlan.tsx line 17):
```typescript
const ActionPlan: React.FC<Props> = ({ data, user, onDeductCredits, onSaveSolution, ... }) => {
  
  // Splits recommendations by priority
  const criticalRecs = data.recommendations.filter(r => r.priority === 'critical');
  const otherRecs = data.recommendations.filter(r => r.priority !== 'critical');
  
  // Calculates total remaining cost (only for recommendations WITHOUT solutions)
  const totalRemainingCost = useMemo(() => {
    return data.recommendations.reduce((acc, rec) => {
      if (data.generatedSolutions && data.generatedSolutions[rec.id]) 
        return acc;  // ← Already has solution, skip
      return acc + getSolutionCost(rec.priority);
    }, 0);
  }, [data?.recommendations, data?.generatedSolutions]);
  
  // When user clicks "Générer Solution"
  const handleApplyRecommendation = async (rec: Recommendation) => {
    const cost = getSolutionCost(rec.priority);
    
    const success = onDeductCredits(cost);  // Check user has enough credits
    if (!success) return;
    
    // Open modal
    setIsModalOpen(true);
    setIsGeneratingSolution(true);
    
    // Generate solution via AI service
    const solution = await generateSolution(rec, data.url);
    setGeneratedSolution(solution);
    
    // SAVE SOLUTION BACK TO APP STATE
    onSaveSolution(data.timestamp, rec.id, solution);  // line 61
    //                    ^^^^^^^^^^^
    //                    Uses AnalysisResult.timestamp as ID
  };
}
```

---

### 1.5 Displaying Recommendation Status

**RecommendationCard.tsx** shows solution status (line 14):

```typescript
interface Props {
  rec: Recommendation;
  existingSolution?: GeneratedSolution | null;    // ← Passed from ActionPlan
  onView?: (solution: GeneratedSolution) => void;
}

const RecommendationCard = ({ rec, cost, onApply, existingSolution, onView }) => {
  const handleButtonClick = (e) => {
    if (existingSolution && onView) {
      onView(existingSolution);  // ← Show existing solution
    } else {
      onApply(rec);              // ← Generate new solution
    }
  };
  
  return (
    <div>
      {existingSolution ? (
        <Check className="text-purple-300" />     // ← Green check icon
      ) : (
        <Zap className="text-accent" />           // ← Lightning bolt
      )}
      
      <button>
        {existingSolution 
          ? `Voir la solution`                    // ← If exists
          : `Générer Solution (-${cost})`         // ← If not
        }
      </button>
    </div>
  );
};
```

From ActionPlan (line 147-154):
```tsx
<RecommendationCard 
  rec={rec} 
  cost={getSolutionCost(rec.priority)}
  onApply={handleApplyRecommendation}
  existingSolution={data.generatedSolutions?.[rec.id]}  // ← Checks if solution exists
  onView={handleOpenStoredSolution}
/>
```

---

### 1.6 Solution Flow in App.tsx

When `onSaveSolution` is called (App.tsx line 283):

```typescript
const handleSaveSolution = useCallback(
  (analysisId: string, recommendationId: string, solution: GeneratedSolution) => {
    const timestamp = new Date().toISOString();
    const solutionWithMeta = { ...solution, timestamp };

    // 1. Update analysisData (current display)
    setAnalysisData(prev => {
      if (!prev || prev.timestamp !== analysisId) return prev;
      
      return {
        ...prev,
        generatedSolutions: {
          ...(prev.generatedSolutions || {}),
          [recommendationId]: solutionWithMeta      // ← Add solution
        }
      };
    });

    // 2. Update history (persistence)
    setHistory(prevHistory => {
      const index = prevHistory.findIndex(item => item.timestamp === analysisId);
      if (index !== -1) {
        const updatedItem = {
          ...prevHistory[index],
          generatedSolutions: {
            ...(prevHistory[index].generatedSolutions || {}),
            [recommendationId]: solutionWithMeta
          }
        };
        
        // Persist to IndexedDB
        const newHistory = [...prevHistory];
        newHistory[index] = updatedItem;
        db.update(index, updatedItem);
        
        return newHistory;
      }
      return prevHistory;
    });
  }, []
);
```

---

## 2. HOW TO DETERMINE IF ALL RECOMMENDATIONS HAVE GENERATED SOLUTIONS

### 2.1 Simple Check Function

```typescript
// Check if ALL recommendations have solutions
const allSolutionsValidated = (analysis: AnalysisResult): boolean => {
  if (!analysis.recommendations || analysis.recommendations.length === 0) {
    return true;  // No recommendations = technically complete
  }
  
  return analysis.recommendations.every(rec => 
    analysis.generatedSolutions && 
    analysis.generatedSolutions[rec.id]  // Solution must exist for this rec
  );
};

// Count completed vs total
const getValidationProgress = (analysis: AnalysisResult) => {
  const total = analysis.recommendations.length;
  const completed = Object.keys(analysis.generatedSolutions || {}).length;
  return { completed, total, percentage: Math.round((completed / total) * 100) };
};
```

### 2.2 AnalysisResult Type Definition

From types.ts (line 62-75):
```typescript
export interface AnalysisResult {
  url: string;
  timestamp: string;
  globalScore: number;
  categories: Record<string, CategoryScore>; 
  recommendations: Recommendation[];      // ← Array of all recommendations
  aiCitation: { ... };
  robotsTxtData?: RobotsTxtData;
  generatedSolutions?: Record<string, GeneratedSolution & { timestamp: string }>;
  //                    ↑ Key is recommendation.id, value is the solution
}
```

**Key insight**: 
- `generatedSolutions` is a **Record** (object), not an array
- Keys are recommendation IDs
- If a recommendation ID is NOT a key, that solution hasn't been generated
- To verify all are done: check `recommendations.length === Object.keys(generatedSolutions || {}).length`

---

## 3. CURRENT DIRECTORY.TSX INTERFACE & PROPS

### 3.1 DirectoryProps Interface

From Directory.tsx (line 6-13):
```typescript
interface DirectoryProps {
  onBack: () => void;
  onViewEntity: (result: AnalysisResult) => void;
  onNavigateToPlan: () => void;
  user: { 
    name: string; 
    email: string; 
    plan: 'free' | 'pro' | 'enterprise'; 
    credits: number 
  } | null;
  directClientCount: number;
  maxClients: number;
}
```

### 3.2 How Directory is Called (App.tsx line 688-701)

```tsx
{currentView === 'directory' && (
  <Directory
    key={`directory-${refreshKey}`}
    onBack={goBack}
    onViewEntity={handleViewDirectoryEntity}
    onNavigateToPlan={() => {
      setInitialAccountTab('subscription');
      navigateTo('account');
    }}
    user={user}
    directClientCount={monthlyProjectCount}
    maxClients={user ? PLAN_LIMITS[user.plan] : 1}
  />
)}
```

### 3.3 Current Directory Join Flow

Directory.tsx handleJoinIndex() (line 118-127):
```typescript
const handleJoinIndex = () => {
  if (isAtLimit) {
    setShowLimitAlert(true);
    return;
  }
  setViewMode('join');
  setJoinStep('form');
  setAuditScore(null);
  setAuditProgress(0);
};
```

**Currently blocks on**:
- `isAtLimit = directClientCount >= maxClients && user?.plan !== 'enterprise'`
- That's it. No check for Action Plan completion.

---

## 4. THE REQUIREMENT: CONDITIONAL REGISTRATION

**User Story**: 
> The user wants "Rejoindre l'Index" (Join Index) button in Directory to be conditional on ALL improvement solutions from the action plan being validated/generated first.

**What this means**:
1. User should NOT be able to join the Directory unless they've already generated solutions for ALL recommendations
2. If they haven't, show a message like: "Vous devez d'abord compléter votre plan d'action (X/Y solutions générées)" or redirect them to ActionPlan
3. Only allow join if: `allSolutionsValidated(analysisData) === true`

---

## 5. IMPLEMENTATION STRATEGY

### 5.1 What Information Must Flow to Directory

Currently, Directory gets:
- `user` (basic info)
- `directClientCount`, `maxClients` (plan limits)

**Missing**: The current `AnalysisResult` (analysisData)

### 5.2 Pass Current Analysis to Directory

**In App.tsx** (around line 688), add:
```tsx
{currentView === 'directory' && (
  <Directory
    key={`directory-${refreshKey}`}
    onBack={goBack}
    onViewEntity={handleViewDirectoryEntity}
    onNavigateToPlan={...}
    user={user}
    directClientCount={monthlyProjectCount}
    maxClients={user ? PLAN_LIMITS[user.plan] : 1}
    currentAnalysis={analysisData}              // ← ADD THIS
  />
)}
```

### 5.3 Update DirectoryProps Interface

**In Directory.tsx** (line 6-13), add:
```typescript
interface DirectoryProps {
  onBack: () => void;
  onViewEntity: (result: AnalysisResult) => void;
  onNavigateToPlan: () => void;
  user: { ... } | null;
  directClientCount: number;
  maxClients: number;
  currentAnalysis?: AnalysisResult | null;    // ← ADD THIS
}
```

### 5.4 Check Validation Before Join

**In Directory.tsx**, modify handleJoinIndex():
```typescript
const handleJoinIndex = () => {
  // Check 1: Project limit
  if (isAtLimit) {
    setShowLimitAlert(true);
    return;
  }
  
  // Check 2: Action Plan completion (NEW)
  if (!currentAnalysis || !allSolutionsValidated(currentAnalysis)) {
    // Redirect to ActionPlan or show alert
    const { completed, total } = getValidationProgress(currentAnalysis);
    alert(
      `Vous devez d'abord compléter votre plan d'action.\n` +
      `Solutions générées: ${completed}/${total}`
    );
    // Navigate to Projects with ActionPlan tab
    // This would need a prop like onNavigateToActionPlan
    return;
  }
  
  // All checks passed
  setViewMode('join');
  setJoinStep('form');
  ...
};
```

### 5.5 Helper Functions to Add

Create a utility file or add to ActionPlan/Projects:
```typescript
// Shared validation logic
export const allSolutionsValidated = (analysis: AnalysisResult | null | undefined): boolean => {
  if (!analysis?.recommendations?.length) return true;
  
  return analysis.recommendations.every(rec => 
    analysis.generatedSolutions?.[rec.id]
  );
};

export const getValidationProgress = (analysis: AnalysisResult | null | undefined) => {
  if (!analysis?.recommendations?.length) {
    return { completed: 0, total: 0, percentage: 100 };
  }
  
  const total = analysis.recommendations.length;
  const completed = Object.keys(analysis.generatedSolutions || {}).length;
  
  return {
    completed,
    total,
    percentage: Math.round((completed / total) * 100)
  };
};
```

### 5.6 Navigation Prop Required

Directory needs a way to navigate to ActionPlan if validation fails. Add to DirectoryProps:
```typescript
onNavigateToActionPlan?: () => void;  // Prop from App.tsx
```

In App.tsx:
```tsx
<Directory
  ...
  onNavigateToActionPlan={() => {
    navigateTo('projects');
    // Also set the initial tab to action-plan
    setResultsTab('action-plan');
  }}
/>
```

---

## 6. DATA FLOW DIAGRAM

```
┌─────────────────────────────────────────────────────────────────┐
│ App.tsx                                                         │
│                                                                 │
│  history: AnalysisResult[]                                      │
│  analysisData: AnalysisResult | null                            │
│  handleSaveSolution(analysisId, recId, solution)                │
└────────────────┬──────────────────────────────────────────────┬─┘
                 │                                              │
                 │ Pass to Projects                             │
                 │ history[], onSaveSolution                    │
                 │                                              │
        ┌────────▼──────────────────┐                           │
        │ Projects.tsx              │                           │
        │                           │                           │
        │ activeAnalysis:           │                           │
        │ AnalysisResult            │                           │
        │                           │                           │
        │ renderJoinFunnel() ◄──────┼───────────────────┐       │
        │ if activeTab =             │                   │       │
        │ 'action-plan'              │                   │       │
        │                           │                   │       │
        └────────┬──────────────────┘                    │       │
                 │                                       │       │
                 │ Pass activeAnalysis as               │       │
                 │ data prop + onSaveSolution            │       │
                 │                                       │       │
        ┌────────▼─────────────────────────┐            │       │
        │ ActionPlan.tsx                  │            │       │
        │                                 │            │       │
        │ data: AnalysisResult            │            │       │
        │ onSaveSolution callback         │            │       │
        │                                 │            │       │
        │ For each recommendation:        │            │       │
        │  IF exists in data              │            │       │
        │    .generatedSolutions[rec.id]  │            │       │
        │    THEN show ✓                  │            │       │
        │    ELSE show Generate button    │            │       │
        │                                 │            │       │
        │ User clicks "Générer Solution" │            │       │
        │  → generateSolution()           │            │       │
        │  → onSaveSolution()             ├───────┐    │       │
        │                                 │       │    │       │
        └─────────────────────────────────┘       │    │       │
                                                  │    │       │
                ┌─────────────────────────────────┘    │       │
                │                                      │       │
        ┌───────▼──────────────────────────────────┐   │       │
        │ App.tsx handleSaveSolution()             │   │       │
        │                                          │   │       │
        │ setAnalysisData({                        │   │       │
        │   ...prev,                               │   │       │
        │   generatedSolutions: {                  │   │       │
        │     ...prev.generatedSolutions,          │   │       │
        │     [recId]: solution + timestamp        │   │       │
        │   }                                       │   │       │
        │ })                                        │   │       │
        │                                          │   │       │
        │ setHistory([...with update])             │   │       │
        │ db.update(index, updatedItem)            │   │       │
        └──────────────────────────────────────────┘   │       │
                                                       │       │
           ┌───────────────────────────────────────────┘       │
           │                                                    │
        ┌──▼──────────────────────────────────────────────────▼──┐
        │ Directory.tsx  (CURRENT ISSUE)                        │
        │                                                       │
        │ MISSING: analysisData / currentAnalysis              │
        │                                                       │
        │ handleJoinIndex():                                   │
        │   if (isAtLimit) → block                             │
        │   if (!allSolutionsValidated()) → block (NEW)       │
        │   else → proceed to join flow                        │
        │                                                       │
        │ renderJoinFunnel()                                   │
        │   form → audit → plan → payment → success           │
        └──────────────────────────────────────────────────────┘
```

---

## 7. SUMMARY OF CHANGES REQUIRED

| Location | Change | Purpose |
|----------|--------|---------|
| **App.tsx** (line ~688) | Pass `currentAnalysis={analysisData}` to Directory | Provide current AnalysisResult to Directory |
| **App.tsx** (line ~688) | Add `onNavigateToActionPlan` prop | Allow Directory to redirect user to ActionPlan |
| **Directory.tsx** (interface) | Add `currentAnalysis?: AnalysisResult \| null` | Accept current analysis |
| **Directory.tsx** (interface) | Add `onNavigateToActionPlan?: () => void` | Accept navigation callback |
| **Directory.tsx** (line 118) | Add validation check in `handleJoinIndex()` | Block join if ActionPlan incomplete |
| **Directory.tsx or utils** | Add helper functions | `allSolutionsValidated()`, `getValidationProgress()` |
| **Projects.tsx** (optional) | Export helper from ActionPlan | Share validation logic |

---

## 8. EXECUTION CHECKLIST

- [ ] Understand AnalysisResult.generatedSolutions structure
- [ ] Confirm recommendation.id matches generatedSolutions key
- [ ] Create utility functions for validation
- [ ] Pass analysisData to Directory in App.tsx
- [ ] Update DirectoryProps interface
- [ ] Update handleJoinIndex() with validation
- [ ] Add onNavigateToActionPlan callback
- [ ] Test: Verify user cannot join if solutions incomplete
- [ ] Test: Verify user CAN join if all solutions generated
- [ ] Test: Verify redirect works when incomplete
