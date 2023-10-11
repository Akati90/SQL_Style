# SQL_Style
This guide establishes our standards for SQL







  -- Preferred
  SELECT
      id    AS account_id,
      name  AS account_name,
      type  AS account_type,
      ...

  -- vs

  -- Not Preferred
  SELECT
      id,
      name,
      type,
      ...
