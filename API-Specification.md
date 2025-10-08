# TERAFFI API Specification
## RESTful API Design for Partnership Network Platform

### Version: v1.0.0
### Base URL: `https://api.teraffi.com/v1`

---

## Authentication

All API requests require authentication via Bearer token:

```http
Authorization: Bearer {jwt_token}
```

### Authentication Endpoints

#### POST /auth/login
```json
{
  "email": "sarah@example.com",
  "password": "secure_password"
}
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_in": 3600,
  "token_type": "Bearer",
  "member": {
    "id": "mem_123456789",
    "email": "sarah@example.com",
    "profile": {
      "name": "Sarah Bennett",
      "company": "Global Brands Inc",
      "role": "Chief Brand Officer"
    },
    "subscription_tier": "enterprise"
  }
}
```

---

## Member Management

### GET /members/profile
Get current member's complete profile

**Response:**
```json
{
  "id": "mem_123456789",
  "email": "sarah@example.com",
  "profile": {
    "personal_info": {
      "name": "Sarah Bennett",
      "title": "Chief Brand Officer",
      "company": "Global Brands Inc",
      "industry": "Consumer Goods",
      "location": "New York, NY"
    },
    "business_info": {
      "company_size": "5000+",
      "annual_revenue": "$5B+",
      "target_markets": ["millennials", "gen-z", "urban"],
      "brand_values": ["sustainability", "innovation", "authenticity"]
    },
    "goals": [
      {
        "id": "goal_001",
        "type": "market_expansion",
        "description": "Expand into Gen Z demographic",
        "target_date": "2024-12-31",
        "priority": "high",
        "created_at": "2024-08-01T00:00:00Z"
      }
    ],
    "preferences": {
      "partnership_types": ["brand_collaboration", "content_partnership"],
      "budget_range": "$100K-$1M",
      "geographic_focus": ["north_america", "europe"]
    }
  },
  "reputation": {
    "score": 4.8,
    "successful_partnerships": 23,
    "total_partnerships": 25,
    "member_since": "2024-01-15T00:00:00Z"
  },
  "subscription": {
    "tier": "enterprise",
    "features": ["unlimited_searches", "premium_analytics", "dedicated_support"],
    "billing_cycle": "annual",
    "next_billing_date": "2025-01-15T00:00:00Z"
  }
}
```

### PUT /members/profile
Update member profile information

**Request:**
```json
{
  "business_info": {
    "target_markets": ["millennials", "gen-z", "urban", "suburban"],
    "brand_values": ["sustainability", "innovation", "authenticity", "inclusivity"]
  },
  "goals": [
    {
      "type": "revenue_growth",
      "description": "Increase partnership revenue by 40%",
      "target_date": "2024-12-31",
      "priority": "high"
    }
  ]
}
```

---

## Affinity Engine

### POST /affinity/discover
Discover partnership opportunities based on member profile and goals

**Request:**
```json
{
  "query": {
    "what_you_do": "I'm Chief Brand Officer at a major consumer goods company focused on sustainable products for millennials",
    "what_you_want": "I want to find content creators and media properties that align with our sustainability mission and reach Gen Z audiences"
  },
  "filters": {
    "partnership_type": ["content_partnership", "brand_collaboration"],
    "budget_range": {
      "min": 50000,
      "max": 500000
    },
    "geographic_focus": ["north_america"],
    "cultural_alignment": ["sustainability", "social_responsibility"]
  },
  "limit": 20
}
```

**Response:**
```json
{
  "request_id": "req_987654321",
  "processing_time_ms": 1847,
  "results": {
    "total_matches": 156,
    "displayed": 20,
    "recommendations": [
      {
        "match_id": "match_001",
        "partner": {
          "id": "mem_content_creator_001",
          "type": "content_creator",
          "name": "EcoLifestyle Productions",
          "description": "Documentary series focusing on sustainable living and environmental consciousness",
          "audience": {
            "size": 2500000,
            "demographics": {
              "age_primary": "18-34",
              "gender_split": {"female": 65, "male": 35},
              "interests": ["sustainability", "lifestyle", "wellness", "social_responsibility"]
            }
          }
        },
        "affinity_score": 94,
        "match_reasons": [
          "Shared focus on sustainability and environmental responsibility",
          "Target audience overlap: 78% alignment with your Gen Z/Millennial targets",
          "Content format aligns with premium brand positioning",
          "Previous successful partnerships with similar consumer brands"
        ],
        "partnership_potential": {
          "estimated_value": "$150K-$300K",
          "roi_prediction": 4.2,
          "success_probability": 0.87,
          "timeline": "6-8 weeks to launch"
        },
        "cultural_trends": [
          {
            "trend": "eco_conscious_consumption",
            "momentum": 0.89,
            "relevance": "high"
          },
          {
            "trend": "authentic_brand_storytelling", 
            "momentum": 0.76,
            "relevance": "medium"
          }
        ]
      }
    ],
    "cultural_intelligence": {
      "trending_topics": [
        "sustainable consumption",
        "climate action",
        "authentic brand partnerships"
      ],
      "emerging_opportunities": [
        "gen_z_environmental_activism",
        "sustainable_lifestyle_content"
      ]
    }
  }
}
```

### POST /affinity/feedback
Provide feedback on affinity match quality

**Request:**
```json
{
  "match_id": "match_001",
  "feedback": {
    "relevance": 5,
    "interest_level": 4,
    "likelihood_to_pursue": 5,
    "comments": "Excellent match - exactly the type of content partner we're looking for"
  }
}
```

---

## Partnership Management

### GET /partnerships
Get list of partnerships for current member

**Query Parameters:**
- `status`: `active`, `completed`, `pending`, `cancelled`
- `stage`: `discovery`, `negotiation`, `execution`, `performance_tracking`
- `limit`: Number of results (default: 20)
- `offset`: Pagination offset

**Response:**
```json
{
  "partnerships": [
    {
      "id": "part_001",
      "title": "Sustainable Living Documentary Series Partnership",
      "partners": [
        {
          "member_id": "mem_123456789",
          "role": "brand_sponsor",
          "company": "Global Brands Inc"
        },
        {
          "member_id": "mem_content_creator_001", 
          "role": "content_creator",
          "company": "EcoLifestyle Productions"
        }
      ],
      "status": "active",
      "stage": "execution",
      "financial": {
        "total_value": 250000,
        "currency": "USD",
        "payment_structure": "milestone_based",
        "commission_rate": 0.08
      },
      "timeline": {
        "start_date": "2024-09-01T00:00:00Z",
        "end_date": "2024-12-31T23:59:59Z",
        "duration_days": 122
      },
      "performance": {
        "completion_percentage": 45,
        "milestones_completed": 3,
        "milestones_total": 7,
        "current_roi": 2.1
      },
      "created_at": "2024-08-15T10:30:00Z",
      "updated_at": "2024-08-20T14:22:00Z"
    }
  ],
  "pagination": {
    "total": 8,
    "limit": 20,
    "offset": 0,
    "has_more": false
  }
}
```

### POST /partnerships
Create a new partnership

**Request:**
```json
{
  "title": "Spring Campaign Collaboration",
  "partner_id": "mem_content_creator_002",
  "partnership_type": "content_collaboration",
  "financial": {
    "total_value": 75000,
    "currency": "USD",
    "payment_structure": "milestone_based"
  },
  "timeline": {
    "start_date": "2024-10-01T00:00:00Z",
    "end_date": "2024-12-15T23:59:59Z"
  },
  "goals": [
    "Increase brand awareness among Gen Z demographic",
    "Create 5 pieces of authentic branded content",
    "Achieve 2M+ total impressions"
  ],
  "milestones": [
    {
      "title": "Creative Brief Approval",
      "description": "Finalize creative direction and content calendar",
      "due_date": "2024-10-10T00:00:00Z",
      "value": 15000
    },
    {
      "title": "First Content Deliverable",
      "description": "Delivery and approval of first 2 content pieces",
      "due_date": "2024-11-01T00:00:00Z",
      "value": 30000
    }
  ]
}
```

### GET /partnerships/{partnership_id}
Get detailed partnership information

**Response:**
```json
{
  "id": "part_001",
  "title": "Sustainable Living Documentary Series Partnership",
  "description": "6-month content partnership featuring sustainable lifestyle integration",
  "partners": [
    {
      "member_id": "mem_123456789",
      "role": "brand_sponsor",
      "contact_person": "Sarah Bennett",
      "permissions": ["approve_content", "approve_payments", "view_analytics"]
    }
  ],
  "status": "active",
  "stage": "execution",
  "financial": {
    "total_value": 250000,
    "currency": "USD",
    "payment_structure": "milestone_based",
    "payments": [
      {
        "milestone_id": "ms_001",
        "amount": 50000,
        "status": "completed",
        "paid_date": "2024-09-15T00:00:00Z"
      },
      {
        "milestone_id": "ms_002", 
        "amount": 75000,
        "status": "pending_approval",
        "due_date": "2024-10-01T00:00:00Z"
      }
    ],
    "commission": {
      "rate": 0.08,
      "total_amount": 20000,
      "collected": 4000,
      "pending": 16000
    }
  },
  "milestones": [
    {
      "id": "ms_001",
      "title": "Creative Brief and Content Calendar",
      "description": "Detailed creative brief with 6-month content calendar",
      "status": "completed",
      "due_date": "2024-09-15T00:00:00Z",
      "completed_date": "2024-09-12T16:30:00Z",
      "deliverables": [
        {
          "name": "Creative Brief Document",
          "file_url": "https://files.teraffi.com/partnerships/part_001/creative-brief.pdf",
          "uploaded_at": "2024-09-10T14:20:00Z"
        }
      ]
    }
  ],
  "collaboration": {
    "workspace_id": "ws_001",
    "active_members": 8,
    "assets_count": 23,
    "last_activity": "2024-08-20T09:15:00Z"
  },
  "analytics": {
    "performance_score": 4.3,
    "content_pieces_delivered": 3,
    "total_impressions": 1250000,
    "engagement_rate": 0.087,
    "roi_current": 2.1,
    "roi_projected": 3.4
  }
}
```

---

## Creative Collaboration

### GET /collaboration/workspaces
Get list of collaboration workspaces for current member

**Response:**
```json
{
  "workspaces": [
    {
      "id": "ws_001",
      "name": "EcoLifestyle Partnership - Creative Development",
      "partnership_id": "part_001",
      "members": [
        {
          "member_id": "mem_123456789",
          "name": "Sarah Bennett",
          "role": "brand_approver",
          "last_active": "2024-08-20T09:15:00Z"
        },
        {
          "member_id": "mem_content_creator_001",
          "name": "Alex Rivera",
          "role": "creative_lead", 
          "last_active": "2024-08-20T11:42:00Z"
        }
      ],
      "assets_count": 23,
      "pending_approvals": 2,
      "recent_activity": {
        "type": "asset_uploaded",
        "description": "New video treatment uploaded for review",
        "timestamp": "2024-08-20T11:42:00Z",
        "user": "Alex Rivera"
      }
    }
  ]
}
```

### POST /collaboration/assets/upload
Upload creative asset to workspace

**Request (multipart/form-data):**
```
workspace_id: ws_001
asset_type: video_treatment
title: Spring Campaign - Video Treatment v2
description: Updated video treatment incorporating brand feedback
file: [binary file data]
approval_required: true
approvers: ["mem_123456789"]
```

**Response:**
```json
{
  "asset": {
    "id": "asset_001",
    "workspace_id": "ws_001",
    "title": "Spring Campaign - Video Treatment v2",
    "type": "video_treatment",
    "file_url": "https://files.teraffi.com/workspaces/ws_001/video-treatment-v2.pdf",
    "uploaded_by": "mem_content_creator_001",
    "uploaded_at": "2024-08-20T11:42:00Z",
    "approval_status": "pending",
    "approvers": [
      {
        "member_id": "mem_123456789",
        "status": "pending",
        "deadline": "2024-08-25T23:59:59Z"
      }
    ],
    "version": 2,
    "previous_versions": ["asset_000"]
  }
}
```

---

## Analytics & Intelligence

### GET /analytics/dashboard
Get comprehensive analytics dashboard data

**Response:**
```json
{
  "dashboard": {
    "overview": {
      "active_partnerships": 8,
      "total_partnership_value": 1250000,
      "projected_annual_revenue": 450000,
      "partnership_success_rate": 0.89,
      "average_partnership_duration_days": 156
    },
    "performance_metrics": {
      "total_partnerships": 25,
      "completed_partnerships": 17,
      "average_roi": 3.2,
      "member_satisfaction": 4.7,
      "repeat_partnership_rate": 0.68
    },
    "financial_summary": {
      "total_revenue": 875000,
      "commission_paid": 70000,
      "pending_payments": 125000,
      "revenue_growth_rate": 0.34
    },
    "partnership_pipeline": {
      "discovery": 12,
      "negotiation": 4,
      "execution": 8,
      "completion": 3
    },
    "top_performing_partnerships": [
      {
        "partnership_id": "part_003",
        "partner_name": "Sustainable Fashion Week",
        "roi": 4.8,
        "total_value": 180000,
        "status": "completed"
      }
    ],
    "cultural_intelligence": {
      "trending_opportunities": [
        {
          "trend": "sustainable_fashion",
          "growth_rate": 0.23,
          "partnership_potential": "high"
        }
      ],
      "recommended_actions": [
        "Explore partnerships in emerging sustainability trends",
        "Focus on Gen Z content creators for higher engagement"
      ]
    }
  }
}
```

### GET /analytics/partnerships/{partnership_id}
Get detailed analytics for specific partnership

**Response:**
```json
{
  "partnership_analytics": {
    "partnership_id": "part_001",
    "performance_overview": {
      "status": "active",
      "completion_percentage": 45,
      "roi_current": 2.1,
      "roi_projected": 3.4,
      "performance_score": 4.3
    },
    "content_metrics": {
      "pieces_delivered": 3,
      "total_impressions": 1250000,
      "total_engagements": 108750,
      "engagement_rate": 0.087,
      "reach": 892000,
      "brand_mentions": 156
    },
    "audience_insights": {
      "demographics": {
        "age_distribution": {
          "18-24": 0.35,
          "25-34": 0.42,
          "35-44": 0.18,
          "45+": 0.05
        },
        "geographic_reach": [
          {"region": "North America", "percentage": 0.68},
          {"region": "Europe", "percentage": 0.22},
          {"region": "Other", "percentage": 0.10}
        ]
      },
      "sentiment_analysis": {
        "positive": 0.78,
        "neutral": 0.18,
        "negative": 0.04,
        "overall_sentiment": "positive"
      }
    },
    "timeline_performance": [
      {
        "date": "2024-09-01",
        "impressions": 125000,
        "engagements": 10875,
        "reach": 89200
      }
    ],
    "benchmark_comparison": {
      "industry_average_roi": 2.8,
      "industry_average_engagement": 0.064,
      "performance_vs_industry": {
        "roi": "+21%",
        "engagement": "+36%"
      }
    }
  }
}
```

---

## Error Handling

### Standard Error Response Format

```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "The request contains invalid parameters",
    "details": [
      {
        "field": "budget_range.min",
        "message": "Minimum budget must be greater than 0"
      }
    ],
    "request_id": "req_987654321",
    "timestamp": "2024-08-20T14:30:00Z"
  }
}
```

### HTTP Status Codes

- `200` - Success
- `201` - Created
- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not Found
- `409` - Conflict
- `422` - Unprocessable Entity
- `429` - Too Many Requests
- `500` - Internal Server Error

---

## Rate Limiting

API requests are rate limited based on subscription tier:

- **Basic**: 100 requests/hour
- **Pro**: 1,000 requests/hour  
- **Enterprise**: 10,000 requests/hour

Rate limit headers included in responses:
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 987
X-RateLimit-Reset: 1629824400
```

---

## Webhooks

TERAFFI supports webhooks for real-time notifications:

### Webhook Events

```json
{
  "event": "partnership.stage_changed",
  "data": {
    "partnership_id": "part_001",
    "previous_stage": "negotiation",
    "new_stage": "execution",
    "changed_by": "mem_123456789",
    "timestamp": "2024-08-20T14:30:00Z"
  },
  "webhook_id": "wh_001",
  "delivery_attempt": 1
}
```

### Webhook Configuration

```json
{
  "url": "https://your-app.com/webhooks/teraffi",
  "events": [
    "partnership.created",
    "partnership.stage_changed", 
    "milestone.completed",
    "affinity.match_found"
  ],
  "secret": "your_webhook_secret"
}
```

This API specification provides the foundation for building robust integrations with the TERAFFI platform, enabling seamless partnership discovery, management, and optimization.