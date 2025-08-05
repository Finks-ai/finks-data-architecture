# Best Practices for Claude Code Sessions

This guide provides proven strategies for maximizing productivity and success when using Claude Code for the Finks data pipeline implementation.

## 🎯 Session Planning

### Before Starting Any Session

1. **Review Dependencies**
   ```markdown
   - [ ] Previous session handoffs reviewed
   - [ ] Required resources available
   - [ ] Repository cloned and ready
   - [ ] Environment variables set
   - [ ] Dependencies installed
   ```

2. **Prepare Context Prime**
   - Write a clear, specific objective
   - Include relevant background
   - Specify constraints and requirements
   - Reference existing patterns

3. **Set Time Expectations**
   - Add 30% buffer to estimates
   - Plan for testing time
   - Account for debugging
   - Include documentation time

### Optimal Session Structure

```
1. Context Setting (5-10 min)
   - Provide context prime
   - Share relevant files
   - Specify success criteria

2. Implementation (60-80%)
   - Focus on one component
   - Test frequently with uv run pytest
   - Commit working code

3. Testing & Validation (15-20%)
   - Run all tests: uv run pytest
   - Validate success criteria
   - Fix any issues

4. Documentation (5-10%)
   - Update handoff notes
   - Document decisions
   - List next steps
```

## 💻 Working with Claude Code

### Effective Context Primes

**Good Example:**
```
I need to deploy Prefect Server on ECS Fargate.
Previous session created ECS cluster and RDS database.
Use existing VPC (vpc-xxx) and subnets from outputs.json.
Focus on minimal working deployment first.
Server needs to connect to RDS at endpoint: xxx.rds.amazonaws.com
Include health checks and CloudWatch logging.
```

**Poor Example:**
```
Set up Prefect.
```

### Managing Long Sessions

For sessions over 4 hours:

1. **Break into Parts**
   ```
   Part 1: Basic implementation (2 hours)
   Part 2: Testing and debugging (1.5 hours)
   Part 3: Production features (1.5 hours)
   ```

2. **Use Checkpoints**
   - Commit after each working feature
   - Document progress in comments
   - Save outputs for reference

3. **Handle Context Loss**
   - If Claude forgets context, provide summary
   - Reference specific files again
   - Use handoff documentation

### Code Quality Standards

1. **Always Include:**
   - Type hints for Python
   - Error handling
   - Logging statements
   - Unit tests
   - Documentation

2. **Follow Patterns:**
   ```python
   # Consistent error handling
   try:
       result = risky_operation()
   except SpecificError as e:
       logger.error(f"Operation failed: {e}")
       raise CustomError(f"Failed to process: {e}") from e
   ```

3. **Testing Strategy:**
   - Write tests during implementation
   - Test edge cases
   - Include integration tests
   - Mock external services

## 🔄 Recovery Strategies

### When Things Go Wrong

1. **Session Fails to Complete**
   - Commit any working code
   - Document what was attempted
   - Identify blockers
   - Plan recovery approach

2. **Tests Keep Failing**
   - Isolate the failing component
   - Simplify the test case
   - Check assumptions
   - Review error messages carefully

3. **Integration Issues**
   - Verify interface contracts
   - Check credentials and permissions
   - Review networking configuration
   - Use debugging tools

### Common Recovery Patterns

```bash
# Rollback infrastructure
pulumi stack export > backup.json
pulumi destroy -y
pulumi stack import < backup.json

# Debug ECS tasks
aws ecs describe-tasks --cluster <cluster> --tasks <task-arn>
aws logs tail /ecs/<service> --follow

# Test connectivity
docker run --rm -it amazonlinux:2 bash
# Install tools and test from within VPC
```

## 📝 Handoff Documentation

### Effective Handoff Template

```markdown
## Session X Handoff

### Completed ✅
- Main objective achieved
- All tests passing
- Deployed to environment

### Files Modified 📁
- path/to/file1.py - Added authentication
- path/to/file2.yml - Updated configuration

### Key Decisions 🎯
- Used approach X because Y
- Deferred feature Z to Phase 2

### Known Issues ⚠️
- Cold start takes 2 minutes
- Needs optimization for large datasets

### Next Steps 🔗
- Session Y can now proceed
- Need to configure X before Session Z

### Validation Commands 🧪
```bash
# Test deployment
curl https://api.example.com/health

# Check logs
aws logs tail /aws/lambda/function
```
```

## 🚀 Productivity Tips

### 1. Parallel Development

When possible, run multiple sessions:
- Infrastructure in one session
- Application code in another
- Documentation in a third

### 2. Template Library

Build a library of templates:
- Pulumi components
- Prefect flow patterns
- Docker configurations
- CI/CD workflows

### 3. Debugging Efficiently

```python
# Add debug logging
import logging
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

# Use breakpoints
import pdb; pdb.set_trace()

# Time operations
import time
start = time.time()
operation()
logger.info(f"Operation took {time.time() - start:.2f}s")
```

### 4. Environment Management

```bash
# Save environment state
env > session_x_env.txt

# Create session-specific configs
cat > .env.session_x << EOF
SESSION_VAR=value
EOF

# Use direnv for automatic loading
echo "source .env.session_x" > .envrc
```

## 🎓 Learning from Sessions

### Post-Session Review

After each session:
1. What went well?
2. What took longer than expected?
3. What would you do differently?
4. What patterns emerged?

### Building Expertise

- Document solutions to tricky problems
- Create runbooks for complex procedures
- Share learnings with team
- Update estimates based on experience

### Continuous Improvement

```markdown
## Session Retrospective

**Duration**: Estimated 4h, Actual 5.5h

**Challenges**:
- ECS networking took extra time
- Health check configuration was tricky

**Solutions Found**:
- Use ALB health checks, not ECS
- Security group needs explicit egress

**For Next Time**:
- Add 2h buffer for ECS sessions
- Prepare networking diagram first
```

## 🔗 Quick Reference

### Session Checklist

```markdown
## Pre-Session

- [ ] Dependencies verified
- [ ] Context prime prepared
- [ ] Previous handoff reviewed
- [ ] Time allocated with buffer

## During Session

- [ ] Following single objective
- [ ] Testing frequently
- [ ] Committing working code
- [ ] Documenting decisions

## Post-Session

- [ ] All code committed
- [ ] Tests passing
- [ ] Handoff documented
- [ ] Next steps clear
```

### Useful Commands

```bash
# AWS debugging
aws sts get-caller-identity
aws ec2 describe-instances --filters "Name=tag:Project,Values=finks"
aws logs tail <log-group> --follow --since 1h

# Docker debugging
docker ps -a
docker logs <container> --tail 100 -f
docker exec -it <container> /bin/bash

# Pulumi debugging
pulumi stack output -j
pulumi refresh
pulumi preview --diff

# Python debugging with uv
uv run pytest -xvs
uv run python -m pdb script.py
uv run python -m cProfile -s cumulative script.py
```

### Emergency Contacts

When truly stuck:
1. Check AWS service health
2. Review CloudTrail for permission issues
3. Check service quotas
4. Consult AWS documentation
5. Use AWS Support if available

---

Remember: Each session builds on previous work. Take time to do it right, and the later sessions will go more smoothly.