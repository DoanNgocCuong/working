## Tên gọi phổ biến nhất: **Software Design Document (SDD)**

**Software Design Document (SDD)** là tên chuẩn và phổ biến nhất cho tài liệu "từ tổng quan đến chi tiết", được sử dụng rộng rãi trong industry và được chuẩn hóa bởi IEEE Std 1016-2009.[copilot4devops+3](https://copilot4devops.com/solid-software-design-document/)​

## Tại sao SDD là tên đúng cho full-stack design document

**Định nghĩa IEEE**: SDD được thiết kế rõ ràng để bao trùm cả **preliminary design (HLD)** và **detailed design (LLD)** trong một tài liệu duy nhất, phục vụ như "blueprint chi tiết cho implementation" và "primary reference for code development".[wildart.github+2](https://wildart.github.io/MISG5020/standards/SDD_Template.pdf)​

**Scope của SDD theo chuẩn**:

- Stage 1: Overall system architecture + data architecture (HLD)
    
- Stage 2: Detailed data structures + algorithms for the chosen architecture (LLD)[engstandards.lanl+2](https://engstandards.lanl.gov/esm/software/SWDD-template.docx)​
    

**Thực tế sử dụng**: Các tổ chức lớn (IEEE, LANL, NASA) và nhiều template open-source đều dùng tên "SDD" cho tài liệu end-to-end design.[github+2](https://github.com/jam01/SDD-Template)​

## So sánh với "Software Architecture Document"

**Software Architecture Document (SAD)** thường **chỉ tập trung vào high-level architecture**, không bao gồm chi tiết implementation như LLD.[wiki.sei.cmu+3](https://wiki.sei.cmu.edu/confluence/display/SAD/Software+Architecture+Documentation+Template?src=contextnavpagetreemode)​

Phân biệt rõ:

- **Architecture** = high-level structure, components, relationships, patterns[lucidchart+2](https://www.lucidchart.com/blog/software-architecture-vs-design)​
    
- **Design** = cả architecture + detailed module design, algorithms, data structures[stackoverflow+2](https://stackoverflow.com/questions/704855/software-design-vs-software-architecture)​
    

Nếu chỉ dùng tên "Architecture Document", người đọc sẽ **không expect** tìm thấy LLD (class diagrams, pseudocode, detailed logic) trong đó.[geeksforgeeks+1](https://www.geeksforgeeks.org/system-design/difference-between-software-design-and-software-architecture/)​

## Template chuẩn cho SDD

Theo IEEE Std 1016-2009, cấu trúc SDD gồm:

## Core sections (bắt buộc)

1. **Introduction**: Purpose, scope, overview, definitions[github+2](https://github.com/jam01/SDD-Template)​
    
2. **System Overview**: High-level description, context[wildart.github+1](https://wildart.github.io/MISG5020/standards/SDD_Template.pdf)​
    
3. **System Architecture** (HLD):
    
    - Architectural design
        
    - Decomposition description (components, subsystems)
        
    - Design rationale[wvu+2](https://community.wvu.edu/~hhammar/CU/swarch/lecture%20slides/slides%203%20documenting%20sw%20arch/complete%20example%20on%20documenting%20sw%20arch/SAD-OnlineCateringService.doc)​
        
4. **Data Design** (HLD + LLD):
    
    - Data description
        
    - Data dictionary (entities, attributes, types)[engstandards.lanl+1](https://engstandards.lanl.gov/esm/software/SWDD-template.docx)​
        
5. **Component Design** (LLD):
    
    - Detailed design của từng component
        
    - Algorithms, pseudocode
        
    - Class diagrams, method signatures[stackoverflow+2](https://stackoverflow.com/questions/10297869/design-documents-high-level-and-low-level-design-documents)​
        
6. **Human Interface Design**: UI/UX mockups, screen flows[wildart.github+1](https://wildart.github.io/MISG5020/standards/SDD_Template.pdf)​
    
7. **Requirements Matrix**: Traceability giữa requirements và design entities[wvu+1](https://community.wvu.edu/~hhammar/CU/swarch/lecture%20slides/slides%203%20documenting%20sw%20arch/complete%20example%20on%20documenting%20sw%20arch/SAD-OnlineCateringService.doc)​
    
8. **Appendices**: Diagrams, glossary[engstandards.lanl+1](https://engstandards.lanl.gov/esm/software/SWDD-template.docx)​
    

## Template examples có sẵn

**GitHub templates**:

- IEEE 1016-2009 compliant SDD template (Markdown): [github.com/jam01/SDD-Template](https://github.com/jam01/SDD-Template)[github](https://github.com/jam01/SDD-Template)​
    
- Modern SDD template với detailed design sections: [gist.github.com/iamhenry](https://gist.github.com/iamhenry/2dbabd0d59051eae360d8cfa6a2782bd)[gist.github](https://gist.github.com/iamhenry/2dbabd0d59051eae360d8cfa6a2782bd)​
    
- DID-compliant SDD với architecture diagrams: [github.com/VCTLabs](https://github.com/VCTLabs/software_design_description_template)[github](https://github.com/VCTLabs/software_design_description_template/blob/master/README.rst)​
    

**IEEE official template** (PDF/Word):

- LANL Engineering Standards: [engstandards.lanl.gov/esm/software/SWDD-template.docx](https://engstandards.lanl.gov/esm/software/SWDD-template.docx)[engstandards.lanl](https://engstandards.lanl.gov/esm/software/SWDD-template.docx)​
    
- IEEE 1016 full standard: IEEE Std 1016-1998/2009[bilkent+1](http://cs.bilkent.edu.tr/~cagatay/cs413/1016-1998_00741934.pdf)​
    

## Kết luận & khuyến nghị

**Dùng tên**: **"Software Design Document (SDD)"** hoặc **"<System Name> – Software Design Document"**

**Lý do**:

- Được chuẩn hóa bởi IEEE, được industry hiểu rộng rãi[copilot4devops+2](https://copilot4devops.com/solid-software-design-document/)​
    
- Rõ ràng bao gồm cả HLD và LLD trong một tài liệu[docuwriter+2](https://www.docuwriter.ai/posts/sdd-example-document)​
    
- Có nhiều template mẫu, dễ tìm reference[gist.github+2](https://gist.github.com/iamhenry/2dbabd0d59051eae360d8cfa6a2782bd)​
    

**Tránh dùng**:

- "Software Architecture Document" nếu có LLD (vì architecture thường chỉ có HLD)[wiki.sei.cmu+1](https://wiki.sei.cmu.edu/confluence/display/SAD/Software+Architecture+Documentation+Template?src=contextnavpagetreemode)​
    
- "End-to-End Software Design Document" (thừa từ, không phổ biến)[zoolatech](https://zoolatech.com/blog/end-to-end-software-development/)​
    
- "Software Architecture and Design Document" (dài dòng, không chuẩn)
    

**Nếu muốn nhấn mạnh full-stack**, có thể thêm subtitle:

- "Software Design Document (High-Level & Low-Level Design)"
    
- hoặc trong phần Purpose viết: "This SDD covers both architectural design (HLD) and detailed component design (LLD)..."[wildart.github+1](https://wildart.github.io/MISG5020/standards/SDD_Template.pdf)​
    

1. [https://copilot4devops.com/solid-software-design-document/](https://copilot4devops.com/solid-software-design-document/)
2. [https://github.com/jam01/SDD-Template](https://github.com/jam01/SDD-Template)
3. [https://wildart.github.io/MISG5020/standards/SDD_Template.pdf](https://wildart.github.io/MISG5020/standards/SDD_Template.pdf)
4. [https://engstandards.lanl.gov/esm/software/SWDD-template.docx](https://engstandards.lanl.gov/esm/software/SWDD-template.docx)
5. [http://cs.bilkent.edu.tr/~cagatay/cs413/1016-1998_00741934.pdf](http://cs.bilkent.edu.tr/~cagatay/cs413/1016-1998_00741934.pdf)
6. [https://www.scribd.com/doc/242647886/Design-Specification-Doc-IEEE](https://www.scribd.com/doc/242647886/Design-Specification-Doc-IEEE)
7. [https://github.com/VCTLabs/software_design_description_template/blob/master/README.rst](https://github.com/VCTLabs/software_design_description_template/blob/master/README.rst)
8. [https://wiki.sei.cmu.edu/confluence/display/SAD/Software+Architecture+Documentation+Template?src=contextnavpagetreemode](https://wiki.sei.cmu.edu/confluence/display/SAD/Software+Architecture+Documentation+Template?src=contextnavpagetreemode)
9. [https://www.voa.va.gov/DocumentView.aspx?DocumentID=188](https://www.voa.va.gov/DocumentView.aspx?DocumentID=188)
10. [https://www.lucidchart.com/blog/software-architecture-vs-design](https://www.lucidchart.com/blog/software-architecture-vs-design)
11. [https://www.geeksforgeeks.org/system-design/difference-between-software-design-and-software-architecture/](https://www.geeksforgeeks.org/system-design/difference-between-software-design-and-software-architecture/)
12. [https://stackoverflow.com/questions/704855/software-design-vs-software-architecture](https://stackoverflow.com/questions/704855/software-design-vs-software-architecture)
13. [https://testrigor.com/blog/system-design-vs-software-architecture/](https://testrigor.com/blog/system-design-vs-software-architecture/)
14. [https://community.wvu.edu/~hhammar/CU/swarch/lecture%20slides/slides%203%20documenting%20sw%20arch/complete%20example%20on%20documenting%20sw%20arch/SAD-OnlineCateringService.doc](https://community.wvu.edu/~hhammar/CU/swarch/lecture%20slides/slides%203%20documenting%20sw%20arch/complete%20example%20on%20documenting%20sw%20arch/SAD-OnlineCateringService.doc)
15. [https://stackoverflow.com/questions/10297869/design-documents-high-level-and-low-level-design-documents](https://stackoverflow.com/questions/10297869/design-documents-high-level-and-low-level-design-documents)
16. [https://gist.github.com/iamhenry/2dbabd0d59051eae360d8cfa6a2782bd](https://gist.github.com/iamhenry/2dbabd0d59051eae360d8cfa6a2782bd)
17. [https://www.docuwriter.ai/posts/sdd-example-document](https://www.docuwriter.ai/posts/sdd-example-document)
18. [https://zoolatech.com/blog/end-to-end-software-development/](https://zoolatech.com/blog/end-to-end-software-development/)
19. [https://www.ecs.csun.edu/~rlingard/COMP684/Example2SoftArch.htm](https://www.ecs.csun.edu/~rlingard/COMP684/Example2SoftArch.htm)
20. [https://github.com/bflorat/architecture-document-template](https://github.com/bflorat/architecture-document-template)
21. [https://bit.ai/templates/software-design-document-template](https://bit.ai/templates/software-design-document-template)
22. [https://github.com/joelparkerhenderson/architecture-decision-record](https://github.com/joelparkerhenderson/architecture-decision-record)
23. [https://www.multiplayer.app/system-architecture/software-design-document-template/](https://www.multiplayer.app/system-architecture/software-design-document-template/)
24. [https://document360.com/blog/software-design-document/](https://document360.com/blog/software-design-document/)
25. [https://ccis.ksu.edu.sa/sites/ccis.ksu.edu.sa/files/attach/project_i-final-report-template.doc](https://ccis.ksu.edu.sa/sites/ccis.ksu.edu.sa/files/attach/project_i-final-report-template.doc)
26. [https://blog.cm-dm.com/public/Templates/system-architecture-template.doc](https://blog.cm-dm.com/public/Templates/system-architecture-template.doc)
27. [https://softwaredominos.com/home/software-design-development-articles/high-level-solution-design-documents-what-is-it-and-when-do-you-need-one/](https://softwaredominos.com/home/software-design-development-articles/high-level-solution-design-documents-what-is-it-and-when-do-you-need-one/)
28. [https://ccis.ksu.edu.sa/sites/ccis.ksu.edu.sa/files/attach/project_ii-final-report-template_0.doc](https://ccis.ksu.edu.sa/sites/ccis.ksu.edu.sa/files/attach/project_ii-final-report-template_0.doc)
29. [https://www.reddit.com/r/webdev/comments/wow2qr/good_examples_of_software_architecture/](https://www.reddit.com/r/webdev/comments/wow2qr/good_examples_of_software_architecture/)
30. [https://www.studocu.vn/vn/document/truong-dai-hoc-cong-nghiep-thanh-pho-ho-chi-minh/software-architecture/software-architecture-document-template/94501984](https://www.studocu.vn/vn/document/truong-dai-hoc-cong-nghiep-thanh-pho-ho-chi-minh/software-architecture/software-architecture-document-template/94501984)
31. [https://www.youtube.com/watch?v=AHbqU7GYhYA](https://www.youtube.com/watch?v=AHbqU7GYhYA)
32. [https://www.se.rit.edu/~co-operators/SoftwareArchitectureDocumentation.pdf](https://www.se.rit.edu/~co-operators/SoftwareArchitectureDocumentation.pdf)
33. [https://www.imaginarycloud.com/blog/software-architecture-vs-design](https://www.imaginarycloud.com/blog/software-architecture-vs-design)
34. [https://www.linkedin.com/pulse/low-level-design-document-structure-comprehensive-godhandaraman-kzvnc](https://www.linkedin.com/pulse/low-level-design-document-structure-comprehensive-godhandaraman-kzvnc)
35. [https://www.atlassian.com/work-management/knowledge-sharing/documentation/software-design-document](https://www.atlassian.com/work-management/knowledge-sharing/documentation/software-design-document)
36. [https://testrigor.com/blog/high-level-design-hld-vs-low-level-design-lld/](https://testrigor.com/blog/high-level-design-hld-vs-low-level-design-lld/)
37. [https://www.reddit.com/r/SoftwareEngineering/comments/106jk5k/what_is_the_difference_between_architecture/](https://www.reddit.com/r/SoftwareEngineering/comments/106jk5k/what_is_the_difference_between_architecture/)
38. [http://learnprogramingbyluckysir.blogspot.com/2015/11/how-to-write-effective-design-by-using.html](http://learnprogramingbyluckysir.blogspot.com/2015/11/how-to-write-effective-design-by-using.html)
39. [https://www.geeksforgeeks.org/system-design/difference-between-high-level-design-and-low-level-design/](https://www.geeksforgeeks.org/system-design/difference-between-high-level-design-and-low-level-design/)
40. [https://tecnovy.com/en/software-architecture-vs-design](https://tecnovy.com/en/software-architecture-vs-design)
41. [https://complextester.wordpress.com/2012/08/10/making-of-hld-lld/](https://complextester.wordpress.com/2012/08/10/making-of-hld-lld/)
42. [https://www.scaler.com/topics/high-level-design-and-low-level-design/](https://www.scaler.com/topics/high-level-design-and-low-level-design/)
43. [https://dev.to/mohini_chauhan_1403/software-design-vs-software-architecture-stop-mixing-up-these-two-tech-twins-38o2](https://dev.to/mohini_chauhan_1403/software-design-vs-software-architecture-stop-mixing-up-these-two-tech-twins-38o2)
44. [https://getsdeready.com/lld-vs-hld-key-differences-for-sdes/](https://getsdeready.com/lld-vs-hld-key-differences-for-sdes/)
45. [https://funix.edu.vn/chia-se-kien-thuc/thiet-ke-he-thong-cap-cao-hld/](https://funix.edu.vn/chia-se-kien-thuc/thiet-ke-he-thong-cap-cao-hld/)
46. [https://www.theknowledgeacademy.com/blog/software-architecture-vs-design/](https://www.theknowledgeacademy.com/blog/software-architecture-vs-design/)
47. [https://github.com/AdriaticOrg/sdd](https://github.com/AdriaticOrg/sdd)
48. [http://thecreatingexperts.com/wp-content/uploads/2014/06/HLD.doc](http://thecreatingexperts.com/wp-content/uploads/2014/06/HLD.doc)
49. [https://github.com/wepay/design_doc_template](https://github.com/wepay/design_doc_template)
50. [https://github.com/topics/design-document](https://github.com/topics/design-document)
51. [https://senior.ceng.metu.edu.tr/2014/hebelegubelegom/documents/InitialDesignReport.pdf](https://senior.ceng.metu.edu.tr/2014/hebelegubelegom/documents/InitialDesignReport.pdf)
52. [https://github.com/VCTLabs/MIL-STD-498](https://github.com/VCTLabs/MIL-STD-498)
53. [https://ieeexplore.ieee.org/iel1/2822/6884/00278258.pdf](https://ieeexplore.ieee.org/iel1/2822/6884/00278258.pdf)
54. [https://www.reddit.com/r/networking/comments/bhnp79/hlds_and_llds_documentation_templatesexamples/](https://www.reddit.com/r/networking/comments/bhnp79/hlds_and_llds_documentation_templatesexamples/)
55. [https://github.com/imayobrown/DesignDocumentTemplates](https://github.com/imayobrown/DesignDocumentTemplates)