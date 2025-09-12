# Tool Library Technical Plan

_Published Tools with Variable Abstraction for Workspace Cloning_

## Executive Summary

This document outlines the technical approach for implementing a curated library of published tools that users can clone to their workspace. The core complexity lies in abstracting variables (especially secrets) during publication while maintaining clear dependency communication to users. This feature will enable tool reusability, reduce setup complexity, and create a marketplace-like experience for tool sharing.

## Architecture Overview

### Core Components

1. **Tool Library Service** - Manages published tools and cloning operations
2. **Variable Abstraction Engine** - Handles variable sanitization and dependency mapping
3. **Tool Template System** - Standardized format for publishable tools
4. **Dependency Resolver** - Maps template variables to workspace variables
5. **Access Control Layer** - Manages public/private tool publishing permissions

## Database Schema Design

### New Tables

```sql
-- Published tools library
CREATE TABLE published_tools (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),

    -- Core Information
    name VARCHAR(255) NOT NULL UNIQUE,
    display_name VARCHAR(255) NOT NULL,
    description TEXT NOT NULL,
    category VARCHAR(50) NOT NULL, -- 'web-search', 'database', 'communication', etc.

    -- Publishing Information
    published_by UUID NOT NULL REFERENCES users(id),
    organization VARCHAR(255), -- Optional organization name
    version VARCHAR(20) NOT NULL DEFAULT '1.0.0',
    is_public BOOLEAN DEFAULT true,
    is_verified BOOLEAN DEFAULT false, -- Admin-verified quality tools

    -- Tool Configuration (template with abstracted variables)
    tool_type VARCHAR(50) NOT NULL CHECK (tool_type IN ('api', 'mcp')),
    configuration_template JSONB NOT NULL, -- Tool config with variable placeholders

    -- Variable Dependencies
    required_variables JSONB NOT NULL, -- Array of required variable definitions
    optional_variables JSONB DEFAULT '[]'::JSONB,

    -- Usage Statistics
    clone_count INTEGER DEFAULT 0,
    rating_average DECIMAL(3,2) DEFAULT 0.00,
    rating_count INTEGER DEFAULT 0,

    -- Metadata
    tags JSONB DEFAULT '[]'::JSONB, -- Search/filtering tags
    documentation_url TEXT,
    example_usage TEXT,
    changelog TEXT,

    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    is_active BOOLEAN DEFAULT true
);

-- Tool cloning history
CREATE TABLE tool_clone_history (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    published_tool_id UUID NOT NULL REFERENCES published_tools(id),
    workspace_id UUID NOT NULL REFERENCES workspaces(id),
    cloned_by UUID NOT NULL REFERENCES users(id),
    cloned_tool_id UUID NOT NULL REFERENCES tools(id), -- The actual tool created
    variable_mappings JSONB NOT NULL, -- Maps template variables to workspace variables
    clone_timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Tool ratings and reviews
CREATE TABLE tool_ratings (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    published_tool_id UUID NOT NULL REFERENCES published_tools(id),
    user_id UUID NOT NULL REFERENCES users(id),
    workspace_id UUID NOT NULL REFERENCES workspaces(id),
    rating INTEGER NOT NULL CHECK (rating >= 1 AND rating <= 5),
    review TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    CONSTRAINT unique_user_tool_rating UNIQUE (published_tool_id, user_id, workspace_id)
);

-- Tool categories for organization
CREATE TABLE tool_categories (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(100) NOT NULL UNIQUE,
    display_name VARCHAR(100) NOT NULL,
    description TEXT,
    icon VARCHAR(50), -- Icon identifier for UI
    sort_order INTEGER DEFAULT 0,
    is_active BOOLEAN DEFAULT true
);
```

### Indexes

```sql
-- Performance indexes
CREATE INDEX idx_published_tools_category ON published_tools(category);
CREATE INDEX idx_published_tools_public_active ON published_tools(is_public, is_active);
CREATE INDEX idx_published_tools_verified ON published_tools(is_verified);
CREATE INDEX idx_published_tools_clone_count ON published_tools(clone_count DESC);
CREATE INDEX idx_published_tools_rating ON published_tools(rating_average DESC);
CREATE INDEX idx_tool_clone_history_workspace ON tool_clone_history(workspace_id);
CREATE INDEX idx_tool_clone_history_published_tool ON tool_clone_history(published_tool_id);

-- Search indexes using GIN for JSONB
CREATE INDEX idx_published_tools_tags ON published_tools USING GIN (tags);
CREATE INDEX idx_published_tools_search ON published_tools USING GIN (
    to_tsvector('english', display_name || ' ' || description || ' ' || COALESCE(tags::text, ''))
);
```

## Variable Abstraction System

### Variable Template Format

```json
{
  "required_variables": [
    {
      "name": "TAVILY_API_KEY",
      "type": "secret",
      "description": "API key for Tavily search service",
      "validation": {
        "pattern": "tvly-[a-zA-Z0-9]{32}",
        "required": true
      },
      "documentation": {
        "how_to_obtain": "Sign up at https://tavily.com and get your API key from the dashboard",
        "pricing_info": "Free tier: 1000 searches/month, Paid: $0.002/search",
        "setup_guide": "https://docs.tavily.com/authentication"
      }
    }
  ],
  "optional_variables": [
    {
      "name": "SEARCH_TIMEOUT",
      "type": "string",
      "description": "Request timeout in seconds",
      "default_value": "30",
      "validation": {
        "pattern": "^[1-9][0-9]*$",
        "required": false
      }
    }
  ]
}
```

### Configuration Template Format

```json
{
  "configuration_template": {
    "endpoint": "https://api.tavily.com/search",
    "method": "POST",
    "headers": {
      "Authorization": "Bearer {{TAVILY_API_KEY}}",
      "Content-Type": "application/json"
    },
    "timeout": "{{SEARCH_TIMEOUT|30}}",
    "request_body": {
      "query": "{{query}}",
      "search_depth": "{{search_depth|basic}}",
      "max_results": "{{max_results|5}}"
    },
    "input_schema": {
      "type": "object",
      "properties": {
        "query": {
          "type": "string",
          "description": "The search query"
        },
        "search_depth": {
          "type": "string",
          "enum": ["basic", "advanced"],
          "description": "Depth of search analysis",
          "default": "basic"
        },
        "max_results": {
          "type": "integer",
          "minimum": 1,
          "maximum": 20,
          "description": "Maximum number of results to return",
          "default": 5
        }
      },
      "required": ["query"]
    }
  }
}
```

## Core Services Implementation

### 1. Tool Library Service

```go
// services/tool_library.go
type ToolLibraryService struct {
    publishedToolRepo interfaces.PublishedToolRepository
    toolRepo          interfaces.ToolRepository
    variableRepo      interfaces.VariablesRepository
    templateEngine    *TemplateEngine
}

// PublishTool publishes a workspace tool to the library
func (s *ToolLibraryService) PublishTool(ctx context.Context, req PublishToolRequest) (*PublishedTool, error) {
    // 1. Validate tool ownership and permissions
    // 2. Extract and abstract variables from tool configuration
    // 3. Create variable dependency documentation
    // 4. Save as published tool template
}

// CloneTool clones a published tool to a workspace
func (s *ToolLibraryService) CloneTool(ctx context.Context, req CloneToolRequest) (*Tool, error) {
    // 1. Fetch published tool template
    // 2. Resolve variable mappings from workspace
    // 3. Validate all required variables are available
    // 4. Create new tool in workspace with resolved configuration
    // 5. Record clone history
}

// BrowseTools lists published tools with filtering and search
func (s *ToolLibraryService) BrowseTools(ctx context.Context, filters BrowseFilters) (*PaginatedToolsResponse, error) {
    // 1. Apply category, search, rating filters
    // 2. Handle public/verified tool visibility
    // 3. Return paginated results with metadata
}
```

### 2. Variable Abstraction Engine

```go
// services/variable_abstraction.go
type VariableAbstractionEngine struct {
    variableExtractor *VariableExtractor
    templateGenerator *TemplateGenerator
}

// AbstractVariables extracts variables from tool configuration and creates template
func (e *VariableAbstractionEngine) AbstractVariables(config map[string]interface{}) (*VariableAbstraction, error) {
    // 1. Deep scan configuration for variable references
    // 2. Classify variables (secret, string, numeric)
    // 3. Generate variable templates with documentation requirements
    // 4. Create sanitized configuration template
    // 5. Validate template can be resolved back to original
}

// ResolveVariables fills template with workspace variable values
func (e *VariableAbstractionEngine) ResolveVariables(template map[string]interface{}, variableMappings map[string]string) (map[string]interface{}, error) {
    // 1. Replace all template placeholders with actual values
    // 2. Apply default values for optional variables
    // 3. Validate resolved configuration
}

// ValidateVariableMapping ensures all required variables are mapped
func (e *VariableAbstractionEngine) ValidateVariableMapping(
    requiredVars []VariableTemplate,
    workspaceVariables []*Variable,
    mappings map[string]string,
) error {
    // 1. Check all required variables are mapped
    // 2. Validate variable types match
    // 3. Apply validation patterns if specified
}
```

### 3. Template Processing

```go
// Template placeholder formats:
// {{VARIABLE_NAME}} - Required variable
// {{VARIABLE_NAME|default_value}} - Optional variable with default
// {{VARIABLE_NAME?}} - Optional variable, empty if not found

type TemplateEngine struct{}

func (t *TemplateEngine) ProcessTemplate(template string, variables map[string]string) (string, error) {
    // 1. Parse template placeholders
    // 2. Replace with actual values or defaults
    // 3. Validate no unresolved required placeholders remain
}

func (t *TemplateEngine) ExtractPlaceholders(template string) ([]VariablePlaceholder, error) {
    // 1. Find all {{...}} patterns
    // 2. Parse variable name, default value, optional flag
    // 3. Return structured placeholder info
}
```

## API Endpoints

### Tool Library Management

```go
// GET /api/library/tools - Browse published tools
// POST /api/library/tools - Publish a tool from workspace
// GET /api/library/tools/{id} - Get published tool details
// PUT /api/library/tools/{id} - Update published tool (owner only)
// DELETE /api/library/tools/{id} - Unpublish tool (owner only)

// GET /api/library/categories - List tool categories
// GET /api/library/tools/{id}/dependencies - Get variable dependencies
// POST /api/workspaces/{workspace_id}/tools/clone/{published_tool_id} - Clone tool to workspace
```

### Tool Library Endpoints Implementation

```go
// handlers/tool_library.go
func (h *ToolLibraryHandlers) BrowsePublishedTools(c *gin.Context) {
    filters := BrowseFilters{
        Category:   c.Query("category"),
        Search:     c.Query("search"),
        Verified:   c.Query("verified") == "true",
        MinRating:  parseFloat(c.Query("min_rating")),
        Tags:       strings.Split(c.Query("tags"), ","),
        Page:       parseInt(c.Query("page"), 1),
        Limit:      parseInt(c.Query("limit"), 20),
    }

    tools, err := h.libraryService.BrowseTools(c.Request.Context(), filters)
    if err != nil {
        c.JSON(http.StatusInternalServerError, ErrorResponse{Error: "Failed to browse tools"})
        return
    }

    c.JSON(http.StatusOK, tools)
}

func (h *ToolLibraryHandlers) CloneToolToWorkspace(c *gin.Context) {
    workspaceID := c.Param("workspace_id")
    publishedToolID := c.Param("published_tool_id")

    var req CloneToolRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, ErrorResponse{Error: "Invalid request"})
        return
    }
    req.WorkspaceID = workspaceID
    req.PublishedToolID = publishedToolID
    req.ClonedBy = c.GetString("userID")

    tool, err := h.libraryService.CloneTool(c.Request.Context(), req)
    if err != nil {
        if errors.Is(err, ErrVariablesMissing) {
            c.JSON(http.StatusBadRequest, ErrorResponse{
                Error: "Missing required variables",
                Code:  "MISSING_VARIABLES",
                Meta:  map[string]interface{}{"missing_variables": req.MissingVariables},
            })
            return
        }
        c.JSON(http.StatusInternalServerError, ErrorResponse{Error: "Failed to clone tool"})
        return
    }

    c.JSON(http.StatusCreated, tool)
}
```

## User Experience Flow

### Tool Publishing Flow

1. **Select Tool to Publish**

   - User navigates to their workspace tools
   - Clicks "Publish Tool" on a configured tool
   - System validates tool is fully functional

2. **Variable Analysis & Documentation**

   - System automatically detects variables in tool configuration
   - Prompts user to document each variable:
     - Description of what the variable is for
     - Instructions on how to obtain the value
     - Whether it's required or optional
     - Any validation patterns or constraints

3. **Publication Metadata**

   - Tool name and description for the library
   - Category selection
   - Tags for searchability
   - Documentation links
   - Version information

4. **Review & Publish**
   - Preview of how tool will appear in library
   - List of variable dependencies clearly shown
   - Confirmation of publication terms
   - Option to make public or organization-only

### Tool Cloning Flow

1. **Browse Library**

   - Search and filter published tools
   - View tool details, ratings, and usage examples
   - See variable dependencies upfront

2. **Variable Mapping**

   - System checks existing workspace variables
   - Shows which required variables are missing
   - Provides guided variable creation for missing dependencies
   - Maps existing variables to tool requirements

3. **Clone Confirmation**
   - Preview of final tool configuration
   - Summary of variable mappings
   - Estimated costs if external API usage involved
   - Clone confirmation and tool activation

## Implementation Phases

### Phase 1: Core Infrastructure (4-6 weeks)

- Database schema implementation
- Variable abstraction engine
- Template processing system
- Basic publishing/cloning API endpoints

### Phase 2: Tool Library Service (3-4 weeks)

- Tool publishing workflow
- Tool cloning with variable resolution
- Browse and search functionality
- Category management

### Phase 3: User Interface (4-5 weeks)

- Library browsing interface
- Tool publishing wizard
- Variable mapping interface
- Tool details and documentation pages

### Phase 4: Advanced Features (3-4 weeks)

- Tool ratings and reviews
- Usage analytics and recommendations
- Tool versioning system
- Organization-scoped tool sharing

### Phase 5: Quality & Marketplace Features (2-3 weeks)

- Admin verification workflow
- Tool quality scoring
- Featured tools curation
- Usage reporting and analytics

## Security Considerations

### Variable Security

- **Secret Abstraction**: Ensure no actual secret values are stored in published tools
- **Validation**: Validate variable patterns to prevent malicious injections
- **Scope Isolation**: Variables only accessible within tool execution context
- **Audit Trails**: Log all tool cloning and variable mappings

### Access Control

- **Publishing Permissions**: Control who can publish public vs organization tools
- **Clone Permissions**: Workspace-level controls for tool cloning
- **Tool Verification**: Admin verification process for public tools
- **Rate Limiting**: Prevent abuse of cloning and publishing APIs

### Data Privacy

- **Configuration Sanitization**: Remove all sensitive data during publishing
- **Workspace Isolation**: Published tools can't access other workspace data
- **Usage Analytics**: Anonymized tool usage statistics only
- **GDPR Compliance**: Right to deletion for published tools

## Testing Strategy

### Unit Tests

- Variable abstraction and template resolution
- Tool configuration validation
- Permission and access control logic
- Database operations and constraints

### Integration Tests

- End-to-end publishing and cloning workflows
- Variable mapping with real workspace data
- API endpoint functionality
- Error handling and edge cases

### Security Tests

- Variable injection prevention
- Access control boundary testing
- Data leak prevention validation
- Malicious tool configuration handling

## Monitoring and Analytics

### Tool Library Metrics

- Publishing rates and success/failure ratios
- Most cloned tools and categories
- Variable mapping success rates
- User engagement with library features

### Quality Metrics

- Tool rating distributions
- Clone success vs. abandonment rates
- Variable dependency complexity analysis
- Support request patterns

### Performance Metrics

- Template resolution performance
- Database query optimization
- API response times
- Cache hit rates for popular tools

## Migration Strategy

### Existing Tool Integration

- Batch analysis of existing workspace tools for publication candidates
- Migration utility for converting tools to template format
- Validation of existing variable usage patterns
- Gradual rollout to power users first

### Backward Compatibility

- Existing tools continue working unchanged
- New library features are additive
- API versioning for gradual feature adoption
- Clear deprecation timeline for any breaking changes

## Success Metrics

### Adoption Metrics

- **Tool Publication Rate**: Number of tools published per month
- **Clone Rate**: Ratio of tool clones to tool views
- **User Engagement**: Active users browsing and using library
- **Time to Value**: Time from tool discovery to successful workspace integration

### Quality Metrics

- **Tool Success Rate**: Percentage of cloned tools that work without issues
- **Variable Mapping Success**: Percentage of clones completed without missing variables
- **User Satisfaction**: Tool ratings and review sentiment
- **Support Reduction**: Decrease in tool setup support requests

This comprehensive technical plan provides a roadmap for implementing a sophisticated tool library system that balances usability with security, enabling tool sharing while protecting sensitive workspace data through intelligent variable abstraction.
