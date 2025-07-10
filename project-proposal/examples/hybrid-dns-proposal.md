# Hybrid DNS Management with AWS Route 53

## Hybrid DNS Management Solution for Enterprise Infrastructure

---

# Executive Summary

Tài liệu này trình bày một giải pháp DNS lai sử dụng AWS Route 53 kết hợp với hệ thống DNS tại chỗ nhằm đảm bảo khả năng phân giải tên miền ổn định, bảo mật và dễ quản lý cho doanh nghiệp.

**Vấn đề:** Hệ thống DNS bị phân mảnh giữa môi trường tại chỗ và cloud gây khó khăn trong quản trị và độ tin cậy thấp.

**Giải pháp:**

- Tích hợp Route 53 Resolver Endpoints (Inbound/Outbound)
- Thiết lập Split-horizon DNS
- Cấu hình Conditional Forwarding
- Giám sát và bảo mật bằng CloudWatch và GuardDuty

**Lợi ích:**

- Phân giải DNS thống nhất
- Cải thiện độ tin cậy và bảo mật
- Giảm thiểu lỗi thủ công và chi phí vận hành

**Chi phí:** Ước tính $70 - $120/tháng  
**Thời gian triển khai:** 2–3 tuần  
**Thành công:** >99.9% uptime, độ trễ DNS < 50ms, cấu hình DR hiệu quả

---

# 1. Problem Statement

## Current Situation

Doanh nghiệp có hệ thống DNS riêng tại chỗ, nhưng ngày càng mở rộng lên AWS và gặp vấn đề đồng bộ hóa phân giải DNS, ảnh hưởng đến hoạt động ứng dụng và khả năng giám sát.

## Key Challenges

- Phân giải DNS không thống nhất giữa on-prem và cloud
- Không có cơ chế dự phòng khi DNS on-prem gặp lỗi
- Giám sát DNS phân tán, khó kiểm soát truy vấn

## Stakeholder Impact

- DevOps khó thiết lập và kiểm tra DNS
- End-user bị gián đoạn truy cập dịch vụ
- Quản trị viên mạng tốn thời gian xử lý lỗi

## Business Consequences

- Downtime dịch vụ dẫn đến mất uy tín
- Tăng thời gian phản hồi sự cố
- Không đáp ứng SLA nếu xảy ra lỗi DNS

---

# 2. Solution Architecture

## Architecture Overview

Hệ thống DNS lai được thiết kế với AWS Route 53 Resolver endpoints giao tiếp với DNS on-prem để xử lý các truy vấn giữa hai môi trường.

## AWS Services Used

- Amazon VPC
- Route 53 Resolver (Inbound, Outbound)
- EC2 (test DNS)
- IAM, CloudWatch Logs, GuardDuty

## Component Design

- 1 VPC gồm public & private subnet
- EC2 để test DNS từ private subnet
- Conditional forwarding cho tên miền nội bộ
- Hosted zones cấu hình Split DNS

## Security Architecture

- Mở cổng 53 UDP/TCP với Security Group
- Sử dụng IAM để phân quyền DNS ops
- Kết hợp GuardDuty & VPC Flow Logs giám sát truy vấn

## Scalability Design

- Hỗ trợ forward nhiều miền (multi-rule)
- Thiết kế multi-AZ, multi-VPC với shared resolver
- Có thể tích hợp với các mô hình hybrid khác như AD Connector

---

# 3. Technical Implementation

## Implementation Phases

1. Tạo VPC, EC2, Resolver Endpoint
2. Cấu hình DNS on-prem và routing
3. Thiết lập forwarding rule, tạo hosted zones
4. Ghi log, giám sát
5. Tối ưu và bàn giao

## Technical Requirements

- AWS Account có quyền Route 53 Resolver
- Kết nối VPN hoặc Direct Connect đến on-prem
- On-prem DNS cho phép truy vấn từ AWS IP

## Development Approach

- Kết hợp sử dụng AWS Console và CloudShell
- Dùng script dig/nslookup để kiểm thử
- Terraform để quản lý nếu triển khai nhiều VPC

## Testing Strategy

- `dig`, `nslookup` từ EC2 đến DNS on-prem
- Test thất bại → chuyển sang resolver dự phòng
- Ghi log truy vấn bằng CloudWatch

## Deployment Plan

- Theo từng giai đoạn nhỏ, validate liên tục
- Rollback bằng cách xóa rule/resolver nếu cần

---

# 4. Timeline & Milestones

### Timeline

| Tuần | Mốc công việc                        |
|------|--------------------------------------|
| 1    | Tạo VPC, EC2, Resolver Endpoint      |
| 1.5  | Cấu hình DNS on-prem và routing      |
| 2    | Kiểm thử và tối ưu                   |
| 2.5  | Ghi log, giám sát, bàn giao          |

### Milestones

- Inbound/Outbound resolver hoạt động
- Rule phân giải chính xác
- DNS query log ghi nhận đầy đủ

### Dependencies

- VPN kết nối on-prem
- Đội hạ tầng cung cấp DNS private zone

### Resource Allocation

- 1 kỹ sư thực hiện từ đầu đến cuối
- 2–3 tuần là đủ xử lý hoàn chỉnh độc lập

---

# 5. Budget Estimation

| Hạng mục         | Chi phí (tháng) |
|------------------|-----------------|
| Resolver         | $50–80          |
| EC2 test         | ~$10            |
| CloudWatch Logs  | $5–10           |
| Khác (log, NAT…) | $5–20           |

**Development Cost:** nội bộ tự triển khai  
**Operational Cost:** monitoring + NAT + logs  
**ROI:** giảm downtime DNS, tăng độ tin cậy phân giải → giảm công sức vận hành

---

# 6. Risk Assessment

| Rủi ro                  | Tác động | Xác suất | Giảm thiểu                          |
|-------------------------|----------|----------|--------------------------------------|
| Sai cấu hình Rule DNS   | Cao      | TB       | Template + rà soát peer              |
| Kết nối VPN thất bại    | Cao      | Thấp     | Test kết nối kỹ trước khi triển khai|
| Chi phí tăng đột ngột   | TB       | TB       | Giám sát bằng CloudWatch Budgets    |
| DNS bị tấn công         | Cao      | Thấp     | GuardDuty + SG chỉ định IP cụ thể    |

---

# 7. Expected Outcomes

## Success Metrics

- DNS uptime > 99.9%
- Tối ưu độ trễ < 50ms
- > 95% DNS query xử lý thành công lần đầu

## Business Benefits

- Đảm bảo truy cập ổn định giữa hybrid workloads
- Tự động hoá failover DNS giúp giảm sự cố

## Technical Improvements

- Kết hợp DNS on-prem & cloud trong 1 kiến trúc chuẩn
- Cho phép ghi log & giám sát truy vấn

## Long-term Value

- Mở rộng dễ dàng với multi-VPC & multi-account
- Có thể dùng cho Active Directory, hybrid apps…

---

# Appendices

## A. Technical Specifications

- VPC CIDR: 10.0.0.0/16
- Subnet: 10.0.0.0/24 (public), 10.0.1.0/24 & 10.0.2.0/24 (private)
- Outbound resolver IP: auto-assigned

## B. Cost Calculations

- Resolver: $0.125/hour/endpoint x 2 x 730h ~ $50–60
- Logs: $0.50/GB ghi + lưu trữ

## C. Architecture Diagrams

- [ ] Đính kèm sơ đồ Route 53 resolver (PNG)
- [ ] Flow DNS query từ EC2 → Outbound → On-prem

## D. References

- https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver.html
- AWS Well-Architected Hybrid Networking whitepaper
- Hands-on: Hybrid DNS Resolver setup

