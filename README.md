import React, { useEffect, useMemo, useState } from "react";

const GithubHub = ({
  githubUsername = "octocat",
  linkedinUrl = "https://www.linkedin.com/in/your-profile",
  email = "you@email.com",
  portfolioContactPath = "/contact", // dacă ai pagină contact în app
}) => {
  const [profile, setProfile] = useState(null);
  const [repos, setRepos] = useState([]);
  const [status, setStatus] = useState({ loading: true, error: "" });

  const githubProfileUrl = useMemo(
    () => `https://github.com/${githubUsername}`,
    [githubUsername]
  );

  useEffect(() => {
    let cancelled = false;

    const load = async () => {
      try {
        setStatus({ loading: true, error: "" });

        const [pRes, rRes] = await Promise.all([
          fetch(`https://api.github.com/users/${githubUsername}`),
          fetch(
            `https://api.github.com/users/${githubUsername}/repos?sort=updated&per_page=6`
          ),
        ]);

        if (!pRes.ok) throw new Error("Nu pot încărca profilul GitHub.");
        if (!rRes.ok) throw new Error("Nu pot încărca repo-urile GitHub.");

        const p = await pRes.json();
        const r = await rRes.json();

        if (cancelled) return;
        setProfile(p);
        setRepos(Array.isArray(r) ? r : []);
        setStatus({ loading: false, error: "" });
      } catch (e) {
        if (cancelled) return;
        setStatus({
          loading: false,
          error: e?.message || "A apărut o eroare.",
        });
      }
    };

    load();
    return () => {
      cancelled = true;
    };
  }, [githubUsername]);

  return (
    <section style={styles.wrap}>
      <div style={styles.header}>
        <h2 style={styles.title}>My GitHub Hub</h2>
        <p style={styles.subtitle}>
          Profil + repo-uri recente, plus linkuri rapide spre LinkedIn & Contact.
        </p>
      </div>

      <div style={styles.grid}>
        {/* Profile card */}
        <div style={styles.card}>
          {status.loading ? (
            <div style={styles.skeleton}>
              <div style={{ ...styles.sk, width: 72, height: 72, borderRadius: 18 }} />
              <div style={{ ...styles.sk, height: 16, width: "60%" }} />
              <div style={{ ...styles.sk, height: 12, width: "40%" }} />
              <div style={{ ...styles.sk, height: 12, width: "90%" }} />
            </div>
          ) : status.error ? (
            <div style={styles.error}>
              <p style={{ margin: 0 }}>{status.error}</p>
              <a href={githubProfileUrl} target="_blank" rel="noreferrer" style={styles.link}>
                Deschide profilul GitHub
              </a>
            </div>
          ) : (
            <>
              <div style={styles.profileTop}>
                <img
                  src={profile?.avatar_url}
                  alt="GitHub avatar"
                  style={styles.avatar}
                />
                <div>
                  <div style={styles.nameRow}>
                    <h3 style={styles.name}>{profile?.name || profile?.login}</h3>
                    <span style={styles.badge}>Live</span>
                  </div>
                  <p style={styles.handle}>@{profile?.login}</p>
                </div>
              </div>

              {profile?.bio && <p style={styles.bio}>{profile.bio}</p>}

              <div style={styles.stats}>
                <div style={styles.stat}>
                  <strong>{profile?.public_repos ?? "-"}</strong>
                  <span>Repos</span>
                </div>
                <div style={styles.stat}>
                  <strong>{profile?.followers ?? "-"}</strong>
                  <span>Followers</span>
                </div>
                <div style={styles.stat}>
                  <strong>{profile?.following ?? "-"}</strong>
                  <span>Following</span>
                </div>
              </div>

              <div style={styles.actions}>
                <a
                  href={githubProfileUrl}
                  target="_blank"
                  rel="noreferrer"
                  style={styles.primaryBtn}
                >
                  View GitHub
                </a>

                <a
                  href={linkedinUrl}
                  target="_blank"
                  rel="noreferrer"
                  style={styles.secondaryBtn}
                >
                  LinkedIn
                </a>

                <a href={`mailto:${email}`} style={styles.secondaryBtn}>
                  Email
                </a>

                {/* dacă ai routing intern, poți schimba cu <Link to=...> */}
                <a href={portfolioContactPath} style={styles.secondaryBtn}>
                  Contact
                </a>
              </div>
            </>
          )}
        </div>

        {/* Repos card */}
        <div style={styles.card}>
          <div style={styles.cardHeader}>
            <h3 style={styles.cardTitle}>Recent repositories</h3>
            <a href={githubProfileUrl} target="_blank" rel="noreferrer" style={styles.link}>
              See all →
            </a>
          </div>

          {status.loading ? (
            <div style={styles.repoList}>
              {Array.from({ length: 6 }).map((_, i) => (
                <div key={i} style={styles.repoItem}>
                  <div style={{ ...styles.sk, height: 14, width: "55%" }} />
                  <div style={{ ...styles.sk, height: 11, width: "85%", marginTop: 10 }} />
                </div>
              ))}
            </div>
          ) : status.error ? (
            <p style={{ margin: 0, opacity: 0.8 }}>
              Nu pot afișa repo-urile acum.
            </p>
          ) : repos.length === 0 ? (
            <p style={{ margin: 0, opacity: 0.8 }}>
              Nu am găsit repo-uri publice.
            </p>
          ) : (
            <div style={styles.repoList}>
              {repos.map((r) => (
                <a
                  key={r.id}
                  href={r.html_url}
                  target="_blank"
                  rel="noreferrer"
                  style={styles.repoItemLink}
                >
                  <div style={styles.repoItem}>
                    <div style={styles.repoTop}>
                      <span style={styles.repoName}>{r.name}</span>
                      <span style={styles.repoMeta}>
                        ⭐ {r.stargazers_count}
                      </span>
                    </div>
                    {r.description && (
                      <p style={styles.repoDesc}>{r.description}</p>
                    )}
                    <div style={styles.repoBottom}>
                      {r.language ? <span style={styles.pill}>{r.language}</span> : null}
                      <span style={styles.repoUpdated}>
                        Updated: {new Date(r.updated_at).toLocaleDateString()}
                      </span>
                    </div>
                  </div>
                </a>
              ))}
            </div>
          )}
        </div>
      </div>
    </section>
  );
};

const styles = {
  wrap: {
    maxWidth: 1100,
    margin: "0 auto",
    padding: "28px 16px",
    fontFamily: "system-ui, -apple-system, Segoe UI, Roboto, Arial",
  },
  header: { marginBottom: 18 },
  title: { margin: 0, fontSize: 28, letterSpacing: -0.3 },
  subtitle: { margin: "6px 0 0", opacity: 0.8 },
  grid: {
    display: "grid",
    gridTemplateColumns: "1fr",
    gap: 14,
  },
  card: {
    background: "rgba(255,255,255,0.04)",
    border: "1px solid rgba(255,255,255,0.10)",
    borderRadius: 18,
    padding: 16,
    boxShadow: "0 12px 30px rgba(0,0,0,0.10)",
    backdropFilter: "blur(10px)",
  },
  cardHeader: {
    display: "flex",
    justifyContent: "space-between",
    alignItems: "baseline",
    gap: 10,
    marginBottom: 10,
  },
  cardTitle: { margin: 0, fontSize: 18 },
  link: { textDecoration: "none" },
  profileTop: { display: "flex", gap: 12, alignItems: "center" },
  avatar: { width: 72, height: 72, borderRadius: 18, objectFit: "cover" },
  nameRow: { display: "flex", alignItems: "center", gap: 8 },
  name: { margin: 0, fontSize: 20 },
  badge: {
    fontSize: 12,
    padding: "3px 8px",
    borderRadius: 999,
    border: "1px solid rgba(255,255,255,0.18)",
    opacity: 0.9,
  },
  handle: { margin: "2px 0 0", opacity: 0.75 },
  bio: { margin: "12px 0 0", opacity: 0.9, lineHeight: 1.4 },
  stats: {
    display: "grid",
    gridTemplateColumns: "repeat(3, 1fr)",
    gap: 10,
    marginTop: 14,
  },
  stat: {
    border: "1px solid rgba(255,255,255,0.10)",
    borderRadius: 14,
    padding: 10,
    display: "flex",
    flexDirection: "column",
    gap: 2,
    alignItems: "center",
  },
  actions: {
    display: "grid",
    gridTemplateColumns: "1fr 1fr",
    gap: 10,
    marginTop: 14,
  },
  primaryBtn: {
    gridColumn: "1 / -1",
    textAlign: "center",
    padding: "10px 12px",
    borderRadius: 14,
    border: "1px solid rgba(255,255,255,0.18)",
    textDecoration: "none",
    fontWeight: 600,
  },
  secondaryBtn: {
    textAlign: "center",
    padding: "10px 12px",
    borderRadius: 14,
    border: "1px solid rgba(255,255,255,0.12)",
    textDecoration: "none",
    opacity: 0.95,
  },
  repoList: { display: "grid", gap: 10 },
  repoItemLink: { textDecoration: "none" },
  repoItem: {
    border: "1px solid rgba(255,255,255,0.10)",
    borderRadius: 14,
    padding: 12,
  },
  repoTop: { display: "flex", justifyContent: "space-between", gap: 10 },
  repoName: { fontWeight: 650 },
  repoMeta: { opacity: 0.8, fontSize: 13 },
  repoDesc: { margin: "8px 0 0", opacity: 0.85, lineHeight: 1.35 },
  repoBottom: {
    display: "flex",
    justifyContent: "space-between",
    gap: 10,
    marginTop: 10,
    alignItems: "center",
  },
  pill: {
    fontSize: 12,
    padding: "4px 10px",
    borderRadius: 999,
    border: "1px solid rgba(255,255,255,0.12)",
    opacity: 0.9,
  },
  repoUpdated: { fontSize: 12, opacity: 0.75 },
  skeleton: { display: "grid", gap: 10 },
  sk: {
    background: "rgba(255,255,255,0.10)",
    borderRadius: 12,
  },
  error: {
    display: "grid",
    gap: 10,
    padding: 6,
  },
};

export default GithubHub;
