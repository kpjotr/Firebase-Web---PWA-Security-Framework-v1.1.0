{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://firebase-security-framework.local/schemas/v1.1.0/handoff_contracts.schema.json",
  "title": "Firebase Web & PWA Security Framework v1.1.0 – Multi-Agent Handoff Contracts Schema (CORE-HANDOFF-001)",
  "description": "Machine-validatable JSON Schema for the three-way multi-agent handoff contracts: PLANNER_TO_CODER, CODER_TO_REVIEWER, and REVIEWER_TO_PLANNER.",
  "type": "object",
  "oneOf": [
    { "$ref": "#/$defs/PlannerToCoderContract" },
    { "$ref": "#/$defs/CoderToReviewerContract" },
    { "$ref": "#/$defs/ReviewerToPlannerContract" }
  ],
  "$defs": {
    "StatusEnum": {
      "type": "string",
      "enum": ["PASS", "CONDITIONAL_PASS", "FAIL", "NOT_VERIFIED"]
    },
    "ReviewIndependenceEnum": {
      "type": "string",
      "enum": ["INDEPENDENT_MODEL", "SEPARATE_AGENT_INSTANCE", "SELF_REVIEWED_L1_ONLY"]
    },
    "ExecutedTestItem": {
      "type": "object",
      "additionalProperties": false,
      "required": ["suite", "command", "result", "passed_count", "failed_count"],
      "properties": {
        "suite": { "type": "string", "minLength": 1 },
        "command": { "type": "string", "minLength": 1 },
        "result": { "type": "string", "enum": ["PASS", "FAIL"] },
        "passed_count": { "type": "integer", "minimum": 0 },
        "failed_count": { "type": "integer", "minimum": 0 }
      }
    },
    "DeferredTestItem": {
      "type": "object",
      "additionalProperties": false,
      "required": ["test_ref", "status", "justification", "target_environment"],
      "properties": {
        "test_ref": { "type": "string", "minLength": 1 },
        "status": { "type": "string", "enum": ["DEFERRED", "NOT_APPLICABLE"] },
        "justification": { "type": "string", "minLength": 1 },
        "target_environment": { "type": "string", "minLength": 1 }
      }
    },
    "PlannerToCoderContract": {
      "type": "object",
      "additionalProperties": false,
      "required": [
        "schema_version",
        "framework_version",
        "handoff_type",
        "task_id",
        "project_context",
        "scope",
        "required_security_modules",
        "security_invariants",
        "authorization_expectations",
        "required_tests",
        "prohibited_actions"
      ],
      "properties": {
        "schema_version": { "type": "string", "const": "1.1.0" },
        "framework_version": { "type": "string", "const": "1.1.0" },
        "handoff_type": { "type": "string", "const": "PLANNER_TO_CODER" },
        "task_id": { "type": "string", "minLength": 1 },
        "project_context": {
          "type": "object",
          "additionalProperties": false,
          "required": ["tier", "onboarding_model"],
          "properties": {
            "tier": { "type": "string", "enum": ["L1", "L2", "L3"] },
            "onboarding_model": { "type": "string", "enum": ["A", "B", "C"] }
          }
        },
        "scope": {
          "type": "object",
          "additionalProperties": false,
          "required": [
            "goal",
            "affected_components",
            "allowed_files_to_modify",
            "concurrency_lock",
            "lock_acquisition_mode"
          ],
          "properties": {
            "goal": { "type": "string", "minLength": 1 },
            "affected_components": {
              "type": "array",
              "minItems": 1,
              "items": { "type": "string", "minLength": 1 }
            },
            "allowed_files_to_modify": {
              "type": "array",
              "minItems": 1,
              "items": { "type": "string", "minLength": 1 }
            },
            "concurrency_lock": {
              "type": "string",
              "enum": ["EXCLUSIVE", "SCOPED"]
            },
            "lock_acquisition_mode": {
              "type": "string",
              "enum": ["TECHNICAL_LOCK", "COORDINATION_RESERVATION"]
            }
          }
        },
        "required_security_modules": {
          "type": "array",
          "minItems": 1,
          "items": {
            "type": "object",
            "additionalProperties": false,
            "required": ["id", "module_version"],
            "properties": {
              "id": { "type": "string", "minLength": 1 },
              "module_version": { "type": "string", "minLength": 1 },
              "templates": {
                "type": "array",
                "items": { "type": "string", "minLength": 1 }
              }
            }
          }
        },
        "security_invariants": {
          "type": "array",
          "minItems": 1,
          "items": {
            "type": "object",
            "additionalProperties": false,
            "required": ["id", "requirement_id", "rule"],
            "properties": {
              "id": { "type": "string", "minLength": 1 },
              "requirement_id": { "type": "string", "minLength": 1 },
              "rule": { "type": "string", "minLength": 1 }
            }
          }
        },
        "authorization_expectations": {
          "type": "array",
          "minItems": 1,
          "items": { "type": "string", "minLength": 1 }
        },
        "required_tests": {
          "type": "object",
          "additionalProperties": false,
          "required": ["positive", "negative"],
          "properties": {
            "positive": {
              "type": "array",
              "minItems": 1,
              "items": { "type": "string", "minLength": 1 }
            },
            "negative": {
              "type": "array",
              "minItems": 1,
              "items": { "type": "string", "minLength": 1 }
            }
          }
        },
        "prohibited_actions": {
          "type": "array",
          "minItems": 1,
          "items": { "type": "string", "minLength": 1 }
        }
      }
    },
    "CoderToReviewerContract": {
      "type": "object",
      "additionalProperties": false,
      "required": [
        "schema_version",
        "framework_version",
        "handoff_type",
        "task_id",
        "commit_or_change_id",
        "modified_files",
        "lock_released",
        "verification_evidence",
        "known_issues_and_open_items",
        "residual_risks",
        "proposed_status"
      ],
      "properties": {
        "schema_version": { "type": "string", "const": "1.1.0" },
        "framework_version": { "type": "string", "const": "1.1.0" },
        "handoff_type": { "type": "string", "const": "CODER_TO_REVIEWER" },
        "task_id": { "type": "string", "minLength": 1 },
        "commit_or_change_id": { "type": "string", "minLength": 1 },
        "modified_files": {
          "type": "array",
          "minItems": 1,
          "items": { "type": "string", "minLength": 1 }
        },
        "lock_released": { "type": "boolean" },
        "verification_evidence": {
          "type": "object",
          "additionalProperties": false,
          "required": ["executed_tests", "not_applicable_or_deferred_tests"],
          "properties": {
            "executed_tests": {
              "type": "array",
              "items": { "$ref": "#/$defs/ExecutedTestItem" }
            },
            "not_applicable_or_deferred_tests": {
              "type": "array",
              "items": { "$ref": "#/$defs/DeferredTestItem" }
            }
          }
        },
        "known_issues_and_open_items": {
          "type": "array",
          "items": { "type": "string" }
        },
        "residual_risks": {
          "type": "array",
          "items": { "type": "string" }
        },
        "requested_review_mode": {
          "$ref": "#/$defs/ReviewIndependenceEnum"
        },
        "proposed_status": {
          "$ref": "#/$defs/StatusEnum"
        }
      }
    },
    "ReviewerToPlannerContract": {
      "type": "object",
      "additionalProperties": false,
      "required": [
        "schema_version",
        "framework_version",
        "handoff_type",
        "task_id",
        "reviewed_commit_or_diff_id",
        "inspected_files",
        "independently_executed_tests",
        "findings",
        "rework_tasks",
        "review_independence",
        "final_status"
      ],
      "properties": {
        "schema_version": { "type": "string", "const": "1.1.0" },
        "framework_version": { "type": "string", "const": "1.1.0" },
        "handoff_type": { "type": "string", "const": "REVIEWER_TO_PLANNER" },
        "task_id": { "type": "string", "minLength": 1 },
        "reviewed_commit_or_diff_id": { "type": "string", "minLength": 1 },
        "inspected_files": {
          "type": "array",
          "minItems": 1,
          "items": { "type": "string", "minLength": 1 }
        },
        "independently_executed_tests": {
          "type": "array",
          "items": { "$ref": "#/$defs/ExecutedTestItem" }
        },
        "findings": {
          "type": "array",
          "items": {
            "type": "object",
            "additionalProperties": false,
            "required": [
              "id",
              "severity",
              "requirement_id",
              "file",
              "description",
              "remediation_required"
            ],
            "properties": {
              "id": { "type": "string", "minLength": 1 },
              "severity": {
                "type": "string",
                "enum": ["CRITICAL", "HIGH", "MEDIUM", "LOW"]
              },
              "requirement_id": { "type": "string", "minLength": 1 },
              "file": { "type": "string", "minLength": 1 },
              "description": { "type": "string", "minLength": 1 },
              "remediation_required": { "type": "boolean" }
            }
          }
        },
        "rework_tasks": {
          "type": "array",
          "items": {
            "type": "object",
            "additionalProperties": false,
            "required": ["task_ref", "requirement_ids", "instruction"],
            "properties": {
              "task_ref": { "type": "string", "minLength": 1 },
              "requirement_ids": {
                "type": "array",
                "minItems": 1,
                "items": { "type": "string", "minLength": 1 }
              },
              "instruction": { "type": "string", "minLength": 1 }
            }
          }
        },
        "review_independence": {
          "$ref": "#/$defs/ReviewIndependenceEnum"
        },
        "final_status": {
          "$ref": "#/$defs/StatusEnum"
        }
      }
    }
  }
}