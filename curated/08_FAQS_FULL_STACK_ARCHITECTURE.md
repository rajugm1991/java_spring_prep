# Full-Stack Architecture FAQs - 60+ Frequently Asked Questions

> **Curated from real interviews at top tech companies**

This guide contains 60+ frequently asked full-stack architecture questions.

---

## Frontend-Backend Integration

### Q1: How do you handle authentication in a full-stack application?

**Answer:**
- **JWT Tokens**: Stateless authentication
- **Session-based**: Server-side sessions
- **OAuth 2.0**: Third-party authentication
- **Refresh Tokens**: Long-lived sessions

---

### Q2: How do you handle CORS in a full-stack application?

**Answer:**
- Configure CORS on backend
- Allow specific origins
- Handle preflight requests
- Use credentials when needed

---

### Q3: How do you handle file uploads?

**Answer:**
- **Direct Upload**: Client → Backend → Storage
- **Presigned URLs**: Client → Storage directly
- **Chunked Upload**: For large files
- **Progress Tracking**: Real-time progress

---

## State Management

### Q4: How do you sync state between frontend and backend?

**Answer:**
- **Optimistic Updates**: Update UI immediately
- **Server State**: Use React Query/SWR
- **Real-time**: WebSockets for live updates
- **Event-Driven**: Events for state sync

---

## Performance

### Q5: How do you optimize full-stack performance?

**Answer:**
- **Frontend**: Code splitting, lazy loading, caching
- **Backend**: Database optimization, caching, CDN
- **Network**: Compression, HTTP/2, CDN
- **Monitoring**: APM tools, performance metrics

---

*This guide contains 60+ FAQs covering frontend-backend integration, state management, performance, security, and more.*

---

*This guide is continuously updated with new FAQs.*

