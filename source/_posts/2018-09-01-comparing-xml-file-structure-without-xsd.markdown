---
layout: post
title: "Comparing XML file Structure without XSD"
date: 2018-09-01 22:05:42 +1000
comments: true
categories: [XML, codesnippets]
keywords: XML, compare xml
description: How to compare XML files without XSD
---
Code snippet for comparing two xml files without using xsd for validating their structure is same ( nodes and arguments should be same. Values of each node/argument can be different).



``` csharp

internal void VerifyMessageHaveSimilarStructureOfTemplate(string inputXml, string templateXml)
        {
            var docA = new XmlDocument();
            var docB = new XmlDocument();

            docA.LoadXml(inputXml);
            docB.LoadXml(templateXml);

            var isDifferent = DoTheyHaveDiferentStructure(docA.ChildNodes, docB.ChildNodes);
            log.Info("Result of Checking for difference of Input xml with template is : " + isDifferent.ToString());


        }


  private bool DoTheyHaveDiferentStructure(XmlNodeList xmlNodeListA, XmlNodeList xmlNodeListB)
        {
            if (xmlNodeListA.Count != xmlNodeListB.Count) return true;

            for (var i = 0; i < xmlNodeListA.Count; i++)
            {
                var nodeA = xmlNodeListA[i];
                var nodeB = xmlNodeListB[i];

                if (nodeA.Attributes == null)
                {
                    if (nodeB.Attributes != null)
                        return true;
                    else
                        continue;
                }

                if (nodeA.Attributes.Count != nodeB.Attributes.Count || nodeA.Name != nodeB.Name) return true;
                List<string> AttributeNameA = new List<string>();
                List<string> AttributeNameB = new List<string>();
                for (var j = 0; j < nodeA.Attributes.Count; j++)
                {
                    AttributeNameA.Add(nodeA.Attributes[j].Name);
                    AttributeNameB.Add(nodeB.Attributes[j].Name);
                    // -- If attribute position should be same, then include below as well
                    //var attrA = nodeA.Attributes[j];
                    //var attrB = nodeB.Attributes[j];

                    //if (attrA.Name != attrB.Name) return true;
                }
                AttributeNameA.Sort();
                AttributeNameB.Sort();
                if(! AttributeNameA.SequenceEqual(AttributeNameB)) return true;

                if (nodeA.HasChildNodes && nodeB.HasChildNodes)
                {
                    return HaveDiferentStructure(nodeA.ChildNodes, nodeB.ChildNodes);
                }
                else
                {
                    return true;
                }
            }
            return false;
        }
        
```